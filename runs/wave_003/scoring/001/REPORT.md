# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary run 001. Evidence is bounded to frozen RECONSTRUCTION.md and PLAN.md; the reconstruction was not modified.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **71.7%** |
| Intent fidelity | **45.1%** |
| Combined quality | **7112** |

**Diagnostic split:**

- System-level fidelity: **67.9%**
- Feature-level fidelity: **38.3%**

**(Planning, fidelity) coordinate:** `71.7, 45.1`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-20T14:26:10Z |
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

|File|Total|Captured|Rate|
|---|---|---|---|
|product_brief.md|6|4|66.7%|
|concepts.md|4|2|50.0%|
|bird_engine.md|22|16|72.7%|
|interactions.md|20|14|70.0%|
|aviary_layout.md|18|9|50.0%|
|accounts_sync.md|18|12|66.7%|
|social_optional.md|10|9|90.0%|
|accessibility_perf.md|18|16|88.9%|
|non_goals.md|4|4|100.0%|

Per-feature detail:

|#|Feature title|File|Captured|Note|
|---|---|---|---|---|
|1|Headline product concept statement|product_brief.md|yes|Scope names the V1 Pocket Aviary product and major surfaces.|
|2|"Feels alive, not robotic" design-philosophy section|product_brief.md|no|Mechanisms imply aliveness, but the plan never states the principle as a design philosophy.|
|3|"Notice, never announce" principle callout|product_brief.md|no|Several announcement refusals appear later, but no explicit principle callout is carried.|
|4|Voice-and-tone guide for product surface (naturalist + matter-of-fact)|product_brief.md|yes|Naturalist prose and matter-of-fact error tone are both specified.|
|5|"What this is not" callout (game/Tamagotchi/social-network framing)|product_brief.md|yes|Non-goals explicitly exclude game, custodial, and social-network mechanics.|
|6|Restraint-over-richness scope statement (start with 2 birds, max 7)|product_brief.md|yes|Single scene, starter two, cap seven are present.|
|7|Glossary of domain terms (bird, call, mood, etc.)|concepts.md|no|No glossary-like domain-term section appears.|
|8|Definition of "presence" (idle attention as interaction)|concepts.md|no|Presence pings exist, but the required visibility/focus/activity conjunction is absent.|
|9|Definition of personality vector vs mood (slow vs fast timescale)|concepts.md|yes|Slow personality drift and faster probabilistic mood state are separated.|
|10|Definition of "settle" as user-initiated session end|concepts.md|yes|Settle is an opt-in session close with evening fade and undo.|
|11|Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity)|bird_engine.md|yes|The birds table enumerates all five traits.|
|12|Personality drift function (low-pass filter)|bird_engine.md|yes|Section 5.2 defines a low-pass monotonic drift function.|
|13|Drift rate calibration (one week measurable, three weeks visible)|bird_engine.md|yes|Calibration targets one week measurable and three weeks visible.|
|14|Personality drift is monotonic toward expressive, never punishing|bird_engine.md|yes|Values never decrease; neglect becomes ambient quietness.|
|15|Mood state (fast-timescale, resets daily-ish)|bird_engine.md|yes|Mood FSM, current_mood, and contextual transitions are specified.|
|16|Mood inputs (recent interactions, time of day, ambient events)|bird_engine.md|yes|Mood transition uses time, weather, interaction spikes, and personality.|
|17|Procedural call grammar (motifs combined at runtime)|bird_engine.md|yes|Motif library and sequence compiler are specified.|
|18|Per-bird call signature (recognizable by ear)|bird_engine.md|yes|Seeded calls remain recognizable and repeatable for the same state.|
|19|Chorus mixing (real chorus, not stacked loops)|bird_engine.md|yes|Ambient chorus and listen-in mix behavior are specified.|
|20|Call timing shaped by personality (vocal-frequency trait)|bird_engine.md|no|Vocal_frequency exists, but call timing is not tied to it.|
|21|Idle micro-motion (preen, scan, head-tilt, shuffle)|bird_engine.md|yes|Idle motions include head tilts, weight shifting, preening, and breathing.|
|22|Mood-shaped idle motion|bird_engine.md|no|Idle motion exists, but the plan does not map moods to motion language.|
|23|Bird species pool for v1 (~6 species)|bird_engine.md|no|Species IDs exist, but no v1 species-pool size is specified.|
|24|Bird naming (user-assigned at adoption; renameable)|bird_engine.md|no|A name field exists, but adoption naming and renaming are not specified.|
|25|Adoption flow (two starter birds auto-selected at signup)|bird_engine.md|yes|Starter state adopts exactly two birds.|
|26|Maximum 7 birds per aviary|bird_engine.md|yes|Cap is explicit in scope and rollout.|
|27|Adding a third+ bird (slow unlock based on aviary age, not score)|bird_engine.md|yes|Bird slots unlock by age thresholds through day 365.|
|28|Personality vector persistence (server-side, never resets)|bird_engine.md|yes|Server is canonical and persists bird personality data.|
|29|Mood persistence across sessions|bird_engine.md|yes|Current_mood is stored server-side and snapshots expose persisted mood.|
|30|Bird-to-bird interaction (calls and reactions)|bird_engine.md|no|The plan has chorus audio, but not bird-to-bird reactions.|
|31|Bird identity stability (stable internal id)|bird_engine.md|yes|Each bird has a stable UUID primary key used in state and events.|
|32|Personality vector exposure (NEVER shown numerically)|bird_engine.md|no|The plan does not forbid numeric trait exposure; snapshots expose trait-like fields.|
|33|Return-greeting on viewer arrival|interactions.md|yes|Return greetings are mood/boldness influenced and staggered.|
|34|Greeting variation by absence length|interactions.md|no|Absence-length variation is not specified.|
|35|Greeting variation by bird boldness (bolder birds greet first)|interactions.md|yes|Greeting is boldness-influenced.|
|36|Greeting stagger (multiple birds do not greet simultaneously)|interactions.md|yes|Greetings are randomly staggered.|
|37|No "Welcome back!" toast or banner|interactions.md|no|No textual welcome ban is stated.|
|38|Listen-in interaction (focus a bird; its call rises in the mix)|interactions.md|yes|Listen-in focuses one bird and raises it in the mix.|
|39|Listen-in mix decay (other birds quiet, do not go silent)|interactions.md|yes|Other birds/background fade to 0.1, not zero.|
|40|Offer interaction (seed, song fragment, still pool)|interactions.md|yes|Seed, song fragment, and still pool offers are named.|
|41|Offer reaction varies by bird mood and curiosity|interactions.md|no|Offer events affect mood, but reaction variation by mood/curiosity is not specified.|
|42|Offer cooldown (per-bird cooldown of a few minutes)|interactions.md|yes|Offers are cooldown-bounded gestures.|
|43|Settle gesture (user-initiated session end; lighting shifts to evening)|interactions.md|yes|Settle sends an event and starts an evening fade.|
|44|Settle is opt-in (closing the tab is also valid; not penalized)|interactions.md|yes|Settle is opt-in; raw tab close has no penalties or warnings.|
|45|Field notebook auto-entries (specific naturalist tone)|interactions.md|yes|Sparse naturalist notebook entries are generated by the tick.|
|46|Field notebook entry frequency (rare; only for noteworthy moments)|interactions.md|yes|Entries are sparse and generated only if conditions are met.|
|47|Field notebook is read-only (user cannot edit entries)|interactions.md|no|No edit endpoint appears, but read-only is not specified.|
|48|Presence accounting (idle attention counted as interaction)|interactions.md|yes|Presence events and duration feed drift calculations.|
|49|Presence accounting requires tab focus + cursor + visibility|interactions.md|no|Visibility appears only for render pause; focus/activity conjunction is absent.|
|50|No streak counter, no "days visited" display|interactions.md|yes|No streaks, calendar green dots, stats dashboards, or counters.|
|51|Background-tab pause (client renders only when visible; sim continues server-side)|interactions.md|yes|Hidden documents pause RAF/WebAudio while the server tick continues.|
|52|Click-anywhere-to-undo for the settle gesture (5s window)|interactions.md|no|An undo window exists, but not click-anywhere or five seconds.|
|53|Single horizontal scene (one screen, no panning)|aviary_layout.md|yes|Single responsive horizontal scene with no panning/scrolling/zooming.|
|54|Three perch zones (front, middle, back) shape proximity to viewer|aviary_layout.md|yes|Front, middle, and back perches are drawn.|
|55|Bird-chosen perch (birds choose perch; user does not place birds)|aviary_layout.md|no|Perch state exists, but bird choice and user non-placement are not specified.|
|56|Day/night cycle tied to user local time|aviary_layout.md|yes|Time of day and night/evening backgrounds are modeled.|
|57|Evening palette shift (warmer hues; calls quieter)|aviary_layout.md|no|Settle uses evening lighting, but general warmer/quiet evening state is absent.|
|58|Night state (most birds settled; one nightjar-like bird active)|aviary_layout.md|no|No night-state bird behavior is specified.|
|59|Ambient weather (rare passing rain; soft wind)|aviary_layout.md|yes|Weather, rain, and wind-like ambient sound are present.|
|60|Weather affects mood (rain dampens vocal frequency)|aviary_layout.md|yes|Weather biases mood transitions; rain increases wary odds.|
|61|Ambient leaf/feather drift motion|aviary_layout.md|yes|Leaves drift; reduced motion disables particles.|
|62|Foreground/background parallax (subtle; not parallax-heavy)|aviary_layout.md|yes|Three parallax layers are specified.|
|63|No UI chrome inside the aviary view (icons live in a thin top bar)|aviary_layout.md|yes|A separate top bar contains chrome above the scene.|
|64|Top bar contents (account, settings, accessibility, field notebook, offer affordance)|aviary_layout.md|yes|Top bar lists account/settings/notebook/offer-style controls.|
|65|Top bar auto-fades when cursor is idle|aviary_layout.md|no|No idle fade behavior is specified.|
|66|Aviary scene loads with motion already in progress|aviary_layout.md|no|Snapshots include animation_pose, but no first-frame/mid-action load rule.|
|67|Loading state is a quiet field, not a spinner|aviary_layout.md|no|No loading-state treatment is specified.|
|68|Empty-aviary state (between adoption flow and first bird arriving)|aviary_layout.md|no|No empty-aviary state is specified.|
|69|Color palette spec (calm, naturalist; avoids saturated UI accent colors)|aviary_layout.md|no|Low-contrast sky appears, but no palette spec or accent-color rule.|
|70|Aviary scene is responsive but never crops a bird out of frame|aviary_layout.md|no|Responsive scene appears, but no never-crop rule.|
|71|Email + magic-link sign-in (no passwords)|accounts_sync.md|yes|Passwordless email magic links are specified.|
|72|Magic link expiry (15 minutes)|accounts_sync.md|yes|15-minute expiry is explicit.|
|73|Single-user accounts (one aviary per account at v1)|accounts_sync.md|yes|The account model implies one aviary per account in v1.|
|74|Synthetic account ID (not email-derived) for internal references|accounts_sync.md|yes|Synthetic UUID account IDs are used as internal references.|
|75|Server-side simulation tick (slow cadence, ~once per minute)|accounts_sync.md|yes|The tick runs on the server once per minute.|
|76|Client pulls state snapshot on visibility|accounts_sync.md|yes|Client pulls snapshots; visibility controls render/audio pausing.|
|77|Client interpolates between snapshots for smooth motion|accounts_sync.md|yes|Client interpolates visual coordinates from snapshots.|
|78|Multi-device sync (state is canonical server-side)|accounts_sync.md|yes|Server-canonical multi-device sync is central.|
|79|Last-write-wins is forbidden for personality state|accounts_sync.md|yes|Clients never send absolute values; server tick is sole writer.|
|80|Conflict resolution: server tick is the only writer of personality drift|accounts_sync.md|yes|Only the server tick applies personality updates.|
|81|Sync conflict surface (account-level errors, matter-of-fact tone)|accounts_sync.md|yes|Matter-of-fact error tone is specified for account-level flows.|
|82|Per-device session token (revocable from settings)|accounts_sync.md|no|Session tokens exist, but per-device revocation is absent.|
|83|Account export (download a JSON snapshot of your aviary)|accounts_sync.md|no|No account-export endpoint or flow appears.|
|84|Account deletion (soft-delete, 30-day grace, then hard-delete)|accounts_sync.md|no|deletion_pending_at exists, but grace and hard-delete are absent.|
|85|No telemetry on per-bird interactions for ML model training|accounts_sync.md|no|Telemetry excludes PII/raw observations, but no ML/per-bird interaction rule.|
|86|Aggregate-only telemetry (counts, latencies; never per-bird state)|accounts_sync.md|yes|Telemetry is anonymized performance data with content boundaries.|
|87|Privacy policy link in account settings|accounts_sync.md|no|No privacy-policy link is specified.|
|88|Email change flow (verify new address before switching)|accounts_sync.md|no|No email-change flow is specified.|
|89|Visit invitations (email-based, opt-in per invite)|social_optional.md|yes|Invitations are created for a guest email.|
|90|Visits default OFF for new accounts|social_optional.md|yes|Visits require explicit invite links; no discoverable default is present.|
|91|Visit is read-only ambient view (no interaction by visitor)|social_optional.md|yes|Guest state is read-only and rejects event submissions.|
|92|Visitor cannot trigger greetings, listen-in, or offers|social_optional.md|yes|Read-only guest sessions cannot submit interaction events.|
|93|No chat, no comments, no avatars during visits|social_optional.md|yes|Social-network non-goals exclude likes, comments, guest avatars, and co-presence.|
|94|No "your friend visited!" notification by default|social_optional.md|yes|Default settings set visit notifications false.|
|95|Visit revocation (host can revoke invite at any time)|social_optional.md|yes|The revoke endpoint immediately revokes invitations.|
|96|Visit log (host can see who visited and when, in account settings)|social_optional.md|no|No visit log is modeled.|
|97|Visitor sees host aviary as it is (no special "show-off" mode)|social_optional.md|yes|Guest endpoint returns the same aviary state payload.|
|98|No leaderboards, no aviary discovery feed, no public aviaries|social_optional.md|yes|Social-network non-goals include no public discovery feeds.|
|99|Screen-reader narration of aviary state (running prose)|accessibility_perf.md|yes|Screen-reader narration is running naturalist prose.|
|100|Narration cadence is slow (no overwhelming the SR)|accessibility_perf.md|yes|Idle narration updates every 45 seconds.|
|101|Narration prose is naturalist, not announcement-style|accessibility_perf.md|yes|Narration is naturalist prose rather than raw status lines.|
|102|Reduced-motion mode (slow cross-fades replace micro-motion)|accessibility_perf.md|yes|Reduced motion replaces animations with cross-fades.|
|103|Reduced-motion mode preserves charm (not a stripped fallback)|accessibility_perf.md|yes|It keeps the same aviary rendered through calmer transitions.|
|104|Captioning toggle for procedural calls (text describes mood)|accessibility_perf.md|yes|Call captions are rendered near birds.|
|105|WCAG AA contrast on all user-copy surfaces|accessibility_perf.md|no|Focus visibility is specified, but not WCAG AA copy contrast.|
|106|Keyboard-only navigation through all interactive surfaces|accessibility_perf.md|yes|Keyboard map covers controls, birds, listen-in, and escape.|
|107|Focus indicators visible against the aviary background|accessibility_perf.md|yes|Custom high-contrast glowing outlines are specified.|
|108|Initial JS bundle <2MB|accessibility_perf.md|yes|The <2MB gzipped bundle budget is explicit.|
|109|Time to first bird visible <500ms target on mid-tier mobile/4G|accessibility_perf.md|yes|The <500ms time-to-first-bird budget is explicit.|
|110|60fps idle motion target on 5-year-old laptop|accessibility_perf.md|yes|60fps on 5-year-old mid-range laptops is specified.|
|111|No memory growth over 30-minute session|accessibility_perf.md|yes|CI measures heap growth over 30 minutes.|
|112|Procedural audio synthesized client-side (no large audio downloads)|accessibility_perf.md|yes|WebAudio synthesis is client-side.|
|113|Audio fallback for browsers without WebAudio (graceful silence + captions)|accessibility_perf.md|yes|Fallback silently catches errors, shows mute status, and enables captions.|
|114|Performance observability (synthetic + RUM, aggregate-only)|accessibility_perf.md|yes|Anonymized performance stats are collected.|
|115|Error budget on simulation-tick latency (alarms if >5s p99)|accessibility_perf.md|yes|p99 tick latency above 5s alerts to pager.|
|116|Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge)|accessibility_perf.md|no|No browser support matrix appears.|
|117|Out of scope: native mobile app|non_goals.md|yes|Native apps are explicitly out of scope.|
|118|Out of scope: gamification (achievements, streaks, scores)|non_goals.md|yes|Gamification is explicitly out of scope.|
|119|Out of scope: Tamagotchi-style mechanics (death, hunger, distress)|non_goals.md|yes|Custodial mechanics are explicitly out of scope.|
|120|Out of scope: social network surfaces (profiles, follows, public feed)|non_goals.md|yes|Social networks are explicitly out of scope.|

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **67.9%**.

|Why ID|Weight|Denominator status|Reconstruction evidence|PLAN grounding|B identified?|PLAN cross-cutting?|Rule without why?|Recovery|Note|
|---|---|---|---|---|---|---|---|---|---|
|S1 - feels-alive-not-robotic|4|included|System intent says server-canonical simulation and gradual change; per-feature Return-Greeting says "living state of the aviary rather than be uniform".|PLAN 2.2 client never computes simulation; PLAN 5.4 procedural call grammar; PLAN 9.1 naturalist SR prose; PLAN 7.3 reduced-motion cross-fades.|no|yes|no|partial|The plan carries several aliveness mechanisms, but the reconstruction never articulates the continuing-place / stale-software consequence.|
|S2 - notice-never-announce|4|included|System intent says observer-focused naturalist experience and "gentle, non-obtrusive events"; no seen-vs-processed rationale.|PLAN 1.2 no streaks/stats/badges; PLAN 11.1 gentle field-notebook offers; accounts settings set visit notifications false.|no|yes|no|partial|Announcement refusal survives in scattered rules, but the central affective distinction is mostly absent.|
|S3 - charm-from-specificity|2|included|System intent: naturalist prose rather than raw status lines; product voice is quiet, matter-of-fact, and descriptive.|PLAN 1.1 Field Notebook naturalist prose; PLAN 9.1 SR narration gives specific scene prose; PLAN 1.2 rejects gamified stats dashboards.|yes|yes|no|full|Specific naturalist description over generic status survives well.|
|S4 - restraint-over-richness|2|included|System intent recovers observer-focused naturalist experience, but not the explicit restraint-over-richness rationale.|PLAN 1.1 single horizontal scene with two starter to seven max birds; PLAN 7.1 top bar outside scene; PLAN 8.2 listen-in fades others rather than solo UI.|no|yes|yes|partial|The implementation rules are present; the depth-over-variety rationale is not named by B.|
|S5 - naturalist-voice-with-system-exception|2|included|System intent: auth error is "Matter-of-fact" while narration is naturalist prose rather than raw status lines.|PLAN 4.1 matter-of-fact tone on error; PLAN 9.1 naturalist SR prose; PLAN 1.1 field notebook naturalist prose.|yes|yes|no|full|The voice split is clearly reconstructed and grounded.|
|S6 - presence-is-real-interaction|4|included|System intent: avoids punishment; close-tab path has "No user penalties or warnings"; tick computes presence duration.|PLAN 5.1 computes accumulated presence duration; PLAN 6.2 settle vs close has no penalties; PLAN 1.2 no custodial/Tamagotchi mechanics.|yes|yes|no|partial|Presence as an input and settle/close equivalence survive, but the precise anti-inflation conjunction is missing.|
|S7 - simulation-runs-server-side|4|included|System intent: server is "single source of truth"; client never runs simulation; clients "NEVER send absolute values".|PLAN 2 server source of truth; PLAN 5.1 tick writes state; PLAN 6.1 clients append events and never set personality values.|yes|yes|no|full|The architecture and sync failure prevention are strongly preserved.|
|S8 - privacy-first-on-bird-data|2|included|System intent: privacy boundaries are systemic for PII and metrics; no per-bird relationship-data rationale.|PLAN 3.1 PII isolation; PLAN 10.2 metrics exclude email hashes, bird names, raw observations; no per-bird ML boundary.|no|no|yes|none|Privacy is reconstructed as PII/observability hygiene, not as protection of the per-bird relationship data.|
|S9 - accessibility-as-first-class-surface|4|included|System intent: accessibility is core, not later; narration, reduced motion, captions, keyboard; accessibility changes merge with visual updates.|PLAN 9 says accessibility is core; PLAN 9.1 naturalist narration; PLAN 7.3 reduced motion cross-fades; PLAN 12.4 same-PR accessibility mitigation.|yes|yes|no|full|First-class accessibility and same-release integration are recovered.|

Multi-layer system-level recovery:

|Why ID|L1|L2|L3|
|---|---|---|---|
|S1|no|yes|no|
|S2|no|yes|no|
|S6|yes|no|yes|
|S7|yes|yes|yes|
|S9|yes|yes|yes|

**Cross-cutting evidence appendix:**

- S1: PLAN 2.2 rendering boundary, 5.4 procedural call grammar, 9.1 naturalist narration, 7.3 reduced-motion cross-fades.
- S2: PLAN 1.2 no streaks/stats/badges, 11.1 gentle non-obtrusive field-notebook offers, 3.2 visit notifications default false.
- S3: PLAN 1.1 field notebook naturalist prose, 9.1 screen-reader naturalist prose, bird names in state/export-like data, non-goal refusal of stats/social comparison.
- S4: PLAN 1.1 single horizontal scene with two-to-seven birds, 7.1 top bar outside scene, 8.2 listen-in mix rather than solo tracks.
- S5: PLAN 4.1 matter-of-fact auth error tone, 9.1 naturalist narration, 1.1 field notebook prose.
- S6: PLAN 5.1 presence duration feeds drift, 6.2 settle-vs-close non-penalty, 1.2 no custodial/Tamagotchi punishment.
- S7: PLAN 2 server source of truth, 5.1 one-minute server tick, 6.1 clients append events and never set state, 2.2 hidden clients do not stop the tick.
- S8: PLAN 3.1 PII isolation and 10.2 telemetry boundaries; count is below the gold per-bird relationship-data bar.
- S9: PLAN 9 accessibility core, 9.1 narration, 7.3 reduced motion, 9.2 captions, 9.3 keyboard/focus, 12.4 same-PR mitigation.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **38.3%**.

Reachable feature-level whys: **30 / 40**.

|Why ID|Feature|Weight|Captured?|Denominator status|Reconstruction evidence|PLAN grounding|Rule without why?|Recovery|Note|
|---|---|---|---|---|---|---|---|---|---|
|F1|presence-definition|4|no|unreachable_excluded|none|none|no|unreachable|Anchor feature not captured; the visibility/focus/activity conjunction is absent.|
|F2|drift-function|4|yes|included|Drift rationale: gradual, monotonic, bounded; one week measurable and three weeks visible.|PLAN 5.2 low-pass formula and calibration target values.|no|partial|Slow filter and calibration survived; Tamagotchi-vs-screensaver failure band did not.|
|F3|drift-monotonic-toward-expressive|4|yes|included|System intent: birds cannot die/fall ill/show distress; neglect results only in ambient quietness.|PLAN 1.2 no custodial mechanics; PLAN 5.2 values never decrease.|no|partial|No-punishment intent survived; the two-week return / mistrust consequence was not reconstructed.|
|F4|procedural-call-grammar|4|yes|included|Call-grammar rationale: repeatable but state-sensitive calls; WebAudio synthesizes naturalistic sounds.|PLAN 5.4 motif library/seeded grammar; PLAN 8 WebAudio synthesis.|no|partial|Procedural synthesis survived, but looped-audio deadness, chorus artifact, and audio-spine cascade were mostly absent.|
|F5|mood-shaped-idle-motion|2|no|unreachable_excluded|none|none|no|unreachable|Idle motion is captured, but mood-shaped idle motion is not.|
|F6|bird-count-cap-7|2|yes|included|Age-based ramping rationale says paced expansion up to seven birds.|PLAN 1.1 between two and seven birds; PLAN 11.1 seventh max slot at day 365.|yes|none|The cap rule survived, but the recognizability/chorus-collapse rationale did not.|
|F7|personality-vector-persistence|4|yes|included|Server-canonical state; rendering boundary says client never computes personality/mood; clients never send absolute values.|PLAN 2.2 snapshot boundary; PLAN 6.1 event flow vs state setting.|no|partial|Canonical persistence and no client ownership survived; deleting-the-known-bird rationale did not.|
|F8|vector-never-shown-numerically|2|no|unreachable_excluded|none|none|no|unreachable|The plan does not forbid numeric trait exposure.|
|F9|return-greeting|4|yes|included|Return-Greeting rationale: greetings reflect the living state rather than be uniform; randomly-staggered and mood/boldness-influenced.|PLAN 1.1 randomly-staggered, mood-and-boldness-influenced greetings.|no|partial|Variation by mood/boldness survived, but absence length and notice-never-announce consequence were absent.|
|F10|no-welcome-back-toast|4|no|unreachable_excluded|none|none|no|unreachable|No textual welcome surface is not specified.|
|F11|settle-is-opt-in|2|yes|included|Settle rationale distinguishes intentional close from raw tab-close with no penalties.|PLAN 6.2 settle sends event; close-tab path has no user penalties or warnings.|no|full|The optional ritual and non-penalty rationale are recovered.|
|F12|field-notebook-prose|4|yes|included|Field Notebook documents events in naturalist voice and tick generates entries only if conditions met.|PLAN 1.1 sparsely generated naturalist prose; PLAN 5.1 conditional notebook generation.|no|partial|Naturalist prose and rarity survived; stock-event-log and read-only/journal distinction did not.|
|F13|presence-accounting|4|yes|included|Tick loop computes accumulated presence duration; no evidence for required three-signal conjunction or anti-inflation rationale.|PLAN 5.1 computes presence duration; PLAN 12.1 caps/union model for open tabs.|yes|none|Presence accounting exists, but the load-bearing precision of the conjunction is absent.|
|F14|no-streak-counter|4|yes|included|Gamification rationale: preserve naturalist, observer-focused character; excludes streaks and green dots.|PLAN 1.2 no streaks, level counters, calendar green dots, stats dashboards, badges.|yes|none|The rule survived without the presence-for-birds-not-counter rationale.|
|F15|scene-loads-with-motion|4|no|unreachable_excluded|none|none|no|unreachable|The first-frame already-in-motion rule is absent.|
|F16|synthetic-account-id|4|yes|included|PII Isolation rationale: prevent PII leakage into logs, caches, and telemetry; internal references use synthetic UUIDs.|PLAN 3.1 encrypted email, HMAC lookup, synthetic UUID account_id.|no|partial|PII isolation survived; impossible-to-retrofit consequence did not.|
|F17|server-side-simulation-tick|4|yes|included|Server tick periodically consumes event log, recalculates state, and conflict avoidance is centralized on the server.|PLAN 5.1 one-minute tick; PLAN 6 multi-device conflicts avoided by server centralization.|no|full|Server tick, sync coherence, and client-divergence avoidance all survived.|
|F18|no-last-write-wins|4|yes|included|Event Flow rationale: clients never send absolute values and only append events for sequential server processing.|PLAN 6.1 clients NEVER send absolute values; server simulation tick consumes events sequentially.|no|partial|Implementation rule survived; the silent deletion failure example did not.|
|F19|sync-conflict-tone|2|yes|included|Product voice and auth initiation are matter-of-fact; no error-context evasiveness rationale for sync conflicts.|PLAN 4.1 matter-of-fact tone on error; sync conflicts are structurally avoided.|yes|none|Tone rule appears, but the gold why about charm feeling evasive in system errors is not recovered for sync conflicts.|
|F20|no-per-bird-ml-telemetry|4|no|unreachable_excluded|none|none|no|unreachable|No ML/per-bird interaction telemetry rule is absent.|
|F21|visit-read-only-ambient|2|yes|included|Visits rationale: limited sharing without changing the host aviary; no co-presence or guest-driven drift; event submissions rejected.|PLAN 1.1 read-only guest sharing; PLAN 4.2 guest state rejects event submissions.|no|full|Observation-not-co-presence and no accidental drift are recovered.|
|F22|no-friend-visited-notification|2|yes|included|System intent says sharing is private and non-invasive; no default friend-visited notification rationale.|PLAN 3.2 settings default visit notification false.|yes|none|Default-off notification rule is present, but the attention-driver rationale is missing.|
|F23|no-leaderboards|2|yes|included|Non-goals exclude public discovery feeds and social-network surfaces; no comparison-product rationale.|PLAN 1.2 no social networks/public discovery; no likes/comments/guest avatars.|yes|none|The refusal survived without the comparison-surface why.|
|F24|sr-narration-running-prose|4|yes|included|Screen-reader narration rationale: server compiles coordinates/weather/time into naturalist prose rather than raw status lines.|PLAN 9.1 role=log; naturalist prose example rather than raw coordinates/status.|no|partial|Running prose survived; same-right-to-feel and wrong-ARIA-feature consequences did not.|
|F25|reduced-motion-charm-preserved|4|yes|included|Reduced-motion rationale: accessibility for prefers-reduced-motion; cross-fades replace animation and particles/parallax are disabled.|PLAN 7.3 reduced-motion mode replaces fly-ins and idle animation with cross-fades.|no|partial|Alternate rendering survived; the explicit not-broken/not-costing-product consequence was not reconstructed.|
|F26|ttfb-500ms|2|yes|included|Time-to-first-bird rationale is simply reaching <500ms with server injection and preload.|PLAN 10.1 <500ms and implementation bullets.|yes|none|The metric survived, not the affective threshold explanation.|
|F27|no-gamification-non-goal|4|yes|included|Gamification rationale: preserve naturalist observer-focused character; no streaks, XP, levels, stats, badges.|PLAN 1.2 gamification non-goals list.|yes|none|The rule is explicit, but the counter/foothold/different-product rationale is not.|
|F28|no-tamagotchi-non-goal|2|yes|included|Custodial rationale: avoid distress and obligation; neglect is ambient quietness and monotonic positive drift.|PLAN 1.2 birds cannot die/fall ill/show distress; no punishing drift.|no|full|Observational-not-custodial and no-punishment rationale survive.|
|F29|starter-birds-not-catalog|2|yes|included|Starter state adopts exactly 2 birds: NOT RECOVERABLE FROM PLAN.|PLAN 11.1 starter state adopts exactly 2 birds.|yes|none|Explicit non-recovery; auto-selection rule survived without meeting-animals-not-catalog rationale.|
|F30|age-based-bird-offers|4|yes|included|Age-Based Ramping rationale: paced expansion up to seven; gentle non-obtrusive field-notebook events.|PLAN 11.1 slots unlock by account age; PLAN 1.2 no scores/tiers/billing.|no|partial|Age-based growth survived; the rejection of reward loops and economy erosion did not.|
|F31|stable-bird-identity|4|yes|included|Bird IDs appear in state/data, but no stable-identity why is reconstructed.|PLAN 3.2 birds.id UUID; state returns bird UUIDs.|yes|none|Stable ID mechanism exists without the remembered-relationship rationale.|
|F32|mood-persists-across-sessions|2|yes|included|Current mood is part of canonical state; no no-neutral-reset rationale.|PLAN 3.2 current_mood; PLAN 4.1 state snapshot returns current_mood.|yes|none|Mood persistence is implied, but the continuity illusion why is not recovered.|
|F33|field-notebook-read-only-observer-record|2|no|unreachable_excluded|none|none|no|unreachable|Read-only notebook is not specified.|
|F34|account-export-relationship-copy|2|no|unreachable_excluded|none|none|no|unreachable|Account export is absent.|
|F35|account-deletion-grace-then-hard-delete|4|no|unreachable_excluded|none|none|no|unreachable|Deletion grace and hard-delete behavior are absent.|
|F36|aggregate-telemetry-boundary|2|yes|included|Anonymized performance statistics rationale: observability without content or identity leakage; metrics exclude email hashes, bird names, raw observations.|PLAN 10.2 anonymized performance statistics and telemetry boundaries.|no|full|Aggregate observability boundary is recovered, though the plan is narrower than the gold per-bird formulation.|
|F37|per-invite-named-sharing|2|yes|included|Invite endpoint rationale: creating private guest access by visitor email.|PLAN 4.2 POST /api/visits/invite accepts visitor_email; no social networks.|yes|none|Per-invite rule survived without the host-control/private-relationship rationale.|
|F38|visit-log-on-demand-transparency|2|no|unreachable_excluded|none|none|no|unreachable|Visit log is absent.|
|F39|visitor-sees-actual-aviary|2|yes|included|Guest state endpoint returns the same visual snapshot as aviary state.|PLAN 4.2 GET /api/visits/:token/state returns same payload structure.|yes|none|Same-state mechanism survived; no show-off-mode betrayal rationale was recovered.|
|F40|narration-cadence-slow|4|yes|included|Narration cadence rationale: steady but non-noisy updates; every 45 seconds idle and immediately on interactions.|PLAN 9.1 updates every 45 seconds during idle and immediately upon user interactions.|no|partial|Slow/non-noisy cadence survived; naturalist-presence vs monitoring consequence did not.|

Multi-layer feature-level recovery:

|Why ID|L1|L2|L3|
|---|---|---|---|
|F1|n/a|n/a|n/a|
|F2|yes|yes|no|
|F3|yes|yes|no|
|F4|yes|no|no|
|F7|yes|no|yes|
|F9|no|yes|no|
|F10|n/a|n/a|n/a|
|F12|yes|no|yes|
|F13|no|no|no|
|F14|no|no|no|
|F15|n/a|n/a|n/a|
|F16|yes|yes|no|
|F17|yes|yes|yes|
|F18|yes|no|yes|
|F20|n/a|n/a|n/a|
|F24|yes|no|no|
|F25|yes|yes|no|
|F27|no|no|no|
|F30|yes|no|no|
|F31|no|no|no|
|F35|n/a|n/a|n/a|
|F40|yes|yes|no|

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From constants |
| Possible total weight | 152 | From constants |
| Reachable gold whys | 39 | S whys always included; F whys included only when captured |
| Excluded unreachable feature whys | 10 | Denominator exclusions |
| Recovered / reachable weight | 55 / 122 | Weighted numerator / denominator |
| Whys with reconstruction evidence | 25 | Included whys with full/partial recovery evidence |
| Whys with PLAN grounding | 39 | Included rows with cited plan grounding |
| rule_without_why cases | 15 | Mechanism survived without gold rationale |
| plan_only_not_reconstructed cases | 0 | None counted separately |
| ungrounded_reconstruction cases | 0 | None counted as clear confabulation |

### 2.5. Failure groupings

|Grouping|Recovered weight|Reachable weight|Recovery rate|
|---|---|---|---|
|Functional whys|20.0|40.0|50.0%|
|Affective whys|35.0|82.0|42.7%|
|Weight 2 whys|13.0|34.0|38.2%|
|Weight 3 whys|42.0|88.0|47.7%|
|System-level whys|19.0|28.0|67.9%|
|Feature-level whys (reachable)|36.0|94.0|38.3%|

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional architecture was strongest where the plan was explicit and repeated: S7 and F17 are full. Affective feature whys often became generic rules; F14, F23, F27, F29, F31, F37, and F39 are clear rule-without-why cases.
- **Weight-3 vs weight-2.** High-weight whys often recovered one or two layers but lost Layer 3 downstream consequences. The reconstruction preserved low-pass/calibration mechanics but dropped failure-mode stakes like screensaver/Tamagotchi collapse and economy erosion.
- **System-level vs feature-level.** System-level fidelity was much higher than feature-level fidelity. The planner preserved broad architecture and accessibility principles better than local relationship rationales.
- **Subdomain patterns.** Server sync, simulation tick, and accessibility implementation were relatively strong. Presence precision, privacy/relationship data, social rationale, and loading/aliveness details were weaker.
- **Evidence-bound effects.** The stricter v06 gate denied many plausible v1-style recoveries because exact why evidence was absent in RECONSTRUCTION even when the PLAN had a clear operational rule.

## 4. Recommendations for v2 hardening

- Keep targeted feature-level headroom whys. F29-F40 exposed meaningful differences between rule capture and rationale preservation.
- Add more prompts or scoring surfaces that test downstream consequences. This run often recovered what to build but not what breaks if the rule is relaxed.
- Preserve the system-level cross-cutting appendix. S1 and S2 show why separating inherited mechanisms from explicitly reconstructed principles matters.
- Consider a distinct privacy subscore or more privacy whys. This run treated privacy as PII hygiene while missing the per-bird relationship-data claim.

## 5. Methodology caveats

- Fresh-context isolation held for this scoring pass: I used only the allowed phase-two files and run 001 artifacts, and did not read PRD, peer slots, or other waves.
- This is a single run, so it has no variance signal.
- Borderline capture calls were leaned inclusive where implementation implied the feature: mood state, single-user account shape, visit default-off, and reduced-motion charm.
- System-level cross-cutting remains subjective; the appendix lists the concrete plan locations used for each decision.
- No clear ungrounded-reconstruction confabulation was counted. Thin claims were scored as partial or rule-without-why instead.
