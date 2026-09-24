# REPORT — CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. This report scores the frozen reconstruction against the gold whys and keeps capture, denominator status and recovery separate.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **97.5%** |
| Intent fidelity | **75.7%** |
| Combined quality | **9726** |

**Diagnostic split:**

- System-level fidelity: **89.3%**
- Feature-level fidelity: **72.6%**

**(Planning, fidelity) coordinate:** `(97.5, 75.7)` — plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-24T07:45:52Z |
| Candidate model | gpt-6-sol |
| Candidate effort | high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **117 / 120** = **97.5%**.

| File | Total | Captured | Rate |
|---|---|---|---|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 17 | 94.4% |
| accounts_sync.md | 18 | 18 | 100.0% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **117** | **97.5%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured by the one-account web aviary contract and v1 scope. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured through continuing server simulation, mid-action load, procedural calls and anti-spinner guidance. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by no textual return welcome, no visit announcement and no participation metrics. |
| 4 | Voice-and-tone guide for product surface | product_brief.md | yes | Captured by naturalist product copy and matter-of-fact system/account copy. |
| 5 | "What this is not" callout | product_brief.md | yes | Captured by explicit rejections of game, Tamagotchi and social-network surfaces. |
| 6 | Restraint-over-richness scope statement | product_brief.md | yes | Captured by two starters, seven-bird cap, one scene and sparse top bar. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | The plan uses the terms but does not call for a glossary or explicit domain-term definition surface. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured by visibility, focus and recent pointer/key activity as the qualifying conjunction. |
| 9 | Definition of personality vector vs mood | concepts.md | yes | Captured by slow trait drift versus persisted fast-timescale mood state. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by settle ending presence, evening overlay and undo handling. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured as five server-only normalized trait values influencing behavior. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by smoothed exposure accumulators, additive updates and slow calibration. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured explicitly in simulation harness targets and acceptance criteria. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by max(0) additive drift and no subtraction for absence. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by a small mood state machine with daily-ish equilibrium movement. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by mood shaped by traits, interaction, local time and weather. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by motif grammars and WebAudio synthesis. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by immutable call identity, seed, pitch contour and rhythmic fingerprint. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by ambient chorus, bird-to-bird responses and listening tests. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by vocal-frequency trait changing call probability and chorus participation. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by pose and low-rate idle actions. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by identity, mood and personality selecting visible actions. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by a coherent pool of about six species. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by owner naming and PATCH rename support. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured by provisioned starter birds and owner naming. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by hard cap and database/service validation. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by age-only adoption eligibility. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by canonical database state and server-only trait values. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by persisted mood and timer across sessions. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by responses, chorus and propagated wary/alert reactions. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by stable UUIDs and migration preservation. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by no ordinary UI raw trait numbers and separate export. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by exactly one greeting bird selected on navigation/return. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by brief-absence glance and long-absence reorientation/call. |
| 35 | Greeting variation by bird boldness | interactions.md | yes | Captured by weighted draw using boldness and warmth. |
| 36 | Greeting stagger | interactions.md | yes | Captured by optional second response with seeded offset. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by explicit no welcome toast and no absence tally. |
| 38 | Listen-in interaction | interactions.md | yes | Captured by listen_start/listen_end and target bird mix ramp. |
| 39 | Listen-in mix decay | interactions.md | yes | Captured by non-target birds retaining an audible floor. |
| 40 | Offer interaction | interactions.md | yes | Captured by seed, song-fragment and still-pool offer events. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by mood/personality-shaped offer reaction branches. |
| 42 | Offer cooldown | interactions.md | yes | Captured by server-enforced few-minute per-bird cooldown. |
| 43 | Settle gesture | interactions.md | yes | Captured by settle event, evening lighting and call attenuation. |
| 44 | Settle is opt-in | interactions.md | yes | Captured by rejection of mandatory goodbye and presence ending on close/hidden/unload. |
| 45 | Field notebook auto-entries | interactions.md | yes | Captured by generated naturalist observations from noteworthy transitions. |
| 46 | Field notebook entry frequency | interactions.md | yes | Captured by sparsity budget around one entry per few days. |
| 47 | Field notebook is read-only | interactions.md | yes | Captured by immutable notebook entries and read-only API. |
| 48 | Presence accounting | interactions.md | yes | Captured by qualifying presence intervals as drift input. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by focus, visibility and recent pointer/key conjunction. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by no participation metric, frequency metric or streak. |
| 51 | Background-tab pause | interactions.md | yes | Captured by stop drawing while hidden while server continues ticking. |
| 52 | Click-anywhere-to-undo for settle gesture | interactions.md | yes | Captured by five-second undo click and idempotent undo command. |
| 53 | Single horizontal scene | aviary_layout.md | yes | Captured by one responsive unscrollable scene. |
| 54 | Three perch zones | aviary_layout.md | yes | Captured by front/middle/back perches. |
| 55 | Bird-chosen perch | aviary_layout.md | yes | Captured by server-evolved perches and no scene customization. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured by stored IANA timezone and local-time color transitions. |
| 57 | Evening palette shift | aviary_layout.md | yes | Captured by settle evening lighting/call attenuation and gradual local-time color. |
| 58 | Night state | aviary_layout.md | yes | borderline: Borderline: nightjar-like nighttime activity is present, but most-birds-settled behavior is not spelled out. |
| 59 | Ambient weather | aviary_layout.md | yes | Captured by rare rain/wind events. |
| 60 | Weather affects mood | aviary_layout.md | yes | Captured by temporary mood/audio effects. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by bounded render-only leaf and feather drift. |
| 62 | Foreground/background parallax | aviary_layout.md | yes | Captured by slow parallax in standard mode. |
| 63 | No UI chrome inside aviary view | aviary_layout.md | yes | Captured by sparse top bar and fading chrome. |
| 64 | Top bar contents | aviary_layout.md | yes | Captured by account/settings, accessibility, notebook and offer icons. |
| 65 | Top bar auto-fades | aviary_layout.md | yes | Captured by fade after pointer stillness and restore on input. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by deterministic phase offsets and mid-action first frame. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by quiet field timeout and no loading animation fallback. |
| 68 | Empty-aviary state | aviary_layout.md | yes | Captured by empty field followed by soft first-bird arrival reserved for adoption. |
| 69 | Color palette spec | aviary_layout.md | no | The plan leaves palette tokens for later and does not specify calm naturalist palette or saturated-accent avoidance. |
| 70 | Aviary scene responsive, never crops a bird | aviary_layout.md | yes | Captured by normalized safe bounds for all seven birds. |
| 71 | Email + magic-link sign-in | accounts_sync.md | yes | Captured by magic-link endpoints and email fields. |
| 72 | Magic link expiry | accounts_sync.md | yes | Captured by single-use 15-minute links. |
| 73 | Single-user accounts | accounts_sync.md | yes | Captured by one aviary per active account. |
| 74 | Synthetic account ID | accounts_sync.md | yes | Captured by synthetic UUIDs and email isolation. |
| 75 | Server-side simulation tick | accounts_sync.md | yes | Captured by roughly one-minute server tick. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by snapshot pulls on load/visibility restore. |
| 77 | Client interpolates between snapshots | accounts_sync.md | yes | Captured by interpolation between authoritative snapshots. |
| 78 | Multi-device sync | accounts_sync.md | yes | Captured by canonical server state and no client personality ownership. |
| 79 | Last-write-wins forbidden for personality state | accounts_sync.md | yes | Captured by server-only writer and append-only events. |
| 80 | Conflict resolution via server tick only writer | accounts_sync.md | yes | Captured by tick consuming ordered event log and writing personality state. |
| 81 | Sync conflict surface matter-of-fact tone | accounts_sync.md | yes | Captured by direct system/account copy and quiet errors. |
| 82 | Per-device session token | accounts_sync.md | yes | Captured by opaque token hash and revocable sessions. |
| 83 | Account export | accounts_sync.md | yes | Captured by owner-only JSON snapshot export. |
| 84 | Account deletion | accounts_sync.md | yes | Captured by 30-day recovery then hard purge. |
| 85 | No telemetry on per-bird interactions for ML | accounts_sync.md | yes | Captured by telemetry pipeline separation and no per-bird state. |
| 86 | Aggregate-only telemetry | accounts_sync.md | yes | Captured by aggregate counters, timings and errors only. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured by account endpoint including privacy-policy link. |
| 88 | Email change flow | accounts_sync.md | yes | Captured by verified email transition. |
| 89 | Visit invitations | social_optional.md | yes | Captured by explicitly addressed invitations. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by opt-in per-invite visit model. |
| 91 | Visit is read-only ambient view | social_optional.md | yes | Captured by visitor snapshot with no owner commands. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by no event endpoint scope and no visitor ledger presence. |
| 93 | No chat, comments, avatars during visits | social_optional.md | yes | Captured by social-surface rejections. |
| 94 | No friend-visited notification by default | social_optional.md | yes | Captured by no visit announcement and opt-in notification only. |
| 95 | Visit revocation | social_optional.md | yes | Captured by revoke endpoint and per-pull grant recheck. |
| 96 | Visit log | social_optional.md | yes | Captured by host on-demand visit log. |
| 97 | Visitor sees host aviary as it is | social_optional.md | yes | Captured by same canonical bird/day/weather projection as host. |
| 98 | No leaderboards, discovery feed, public aviaries | social_optional.md | yes | Captured by public discovery/profile/feed/ranking rejections. |
| 99 | Screen-reader narration of aviary state | accessibility_perf.md | yes | Captured by naturalist narration from snapshot/event model. |
| 100 | Narration cadence is slow | accessibility_perf.md | yes | Captured by 30-60 second idle narration cadence. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by meaningful names/species and no raw labels. |
| 102 | Reduced-motion mode | accessibility_perf.md | yes | Captured by cross-fades and explicit preference handling. |
| 103 | Reduced-motion mode preserves charm | accessibility_perf.md | yes | Captured by same poses/events, mood, calls, notebook and drift. |
| 104 | Captioning toggle for procedural calls | accessibility_perf.md | yes | Captured by explicit mute/caption settings and call captions. |
| 105 | WCAG AA contrast | accessibility_perf.md | yes | Captured by AA contrast for every user-copy surface. |
| 106 | Keyboard-only navigation | accessibility_perf.md | yes | Captured by Tab order, arrows, Enter, Escape and keyboard offers/settle. |
| 107 | Focus indicators visible against aviary background | accessibility_perf.md | yes | Captured by visible focus rings in bright and dim scenes. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by compressed JS under 2 MB. |
| 109 | Time to first bird visible <500ms | accessibility_perf.md | yes | Captured by explicit 500 ms returning-session target. |
| 110 | 60fps idle motion target | accessibility_perf.md | yes | Captured by 60 fps target on five-year-old laptop. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by 30-minute memory acceptance gate. |
| 112 | Procedural audio synthesized client-side | accessibility_perf.md | yes | Captured by WebAudio/AudioWorklet synthesis and no downloads. |
| 113 | Audio fallback for browsers without WebAudio | accessibility_perf.md | yes | Captured by silence plus captions on WebAudio failure. |
| 114 | Performance observability | accessibility_perf.md | yes | Captured by synthetic and aggregate-only RUM measurement. |
| 115 | Error budget on simulation-tick latency | accessibility_perf.md | yes | Captured by tick p99 >5s and backlog alarms. |
| 116 | Browser support matrix | accessibility_perf.md | no | No last-two-major browser support matrix is specified. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by rejecting native clients. |
| 118 | Out of scope: gamification | non_goals.md | yes | Captured by rejecting scores, streaks, badges, levels and related surfaces. |
| 119 | Out of scope: Tamagotchi-style mechanics | non_goals.md | yes | Captured by rejecting hunger, death and distress. |
| 120 | Out of scope: social network surfaces | non_goals.md | yes | Captured by rejecting profiles, comments, chats, feeds, rankings and public discovery. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **89.3%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECON §System: "scene should start as an aviary"; §Per-feature: "first frame is mid-action" and procedural calls reject loops. | PLAN §§2,5,6,7,8: quiet field, server tick, mid-action phase, procedural calls, reduced-motion charm. | yes | yes | no | full | Recovered as continuing server life, varied calls/greetings and anti-loading behavior. |
| S2 - notice-never-announce | 4 | included | RECON §System: "quiet naturalist observation rather than a game or dashboard" and "no textual return welcome". | PLAN §§1,4,6,8: no textual return welcome, no visit announcement, no frequency metrics, no gamification copy. | yes | yes | no | partial | Rules and cumulative refusal survived; the processed-vs-seen affective register was not reconstructed. |
| S3 - charm-from-specificity | 2 | included | RECON §System: "lowercase, present-tense naturalist prose" and rejects "generic status strings" and "trait numbers". | PLAN §§1,5,7: naturalist prose, reviewed templates, names/species/perch behavior, no raw state labels. | yes | yes | no | full | Specific naturalist prose versus generic status language is recovered. |
| S4 - restraint-over-richness | 2 | included | none | PLAN §§1,6,7: two starters, seven-bird cap, one unscrollable scene, sparse top bar, gentle chorus and recognizability tests. | no | yes | no | partial | The plan preserves restraint cross-cuttingly, but B did not elevate it as a system-level why. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECON §System: naturalist prose for product surfaces and "direct matter-of-fact copy" for account/system flows. | PLAN §§1,4,7: product voice is naturalist; settings/errors/account flows use matter-of-fact labels and errors. | yes | yes | no | full | The voice split and its system-surface exception are explicit. |
| S6 - presence-is-real-interaction | 4 | included | RECON §System: presence is "a validated input for slow character drift" and "not a public engagement counter". | PLAN §5: focus/visibility/recent input conjunction, drift integrity, no open-tab inference, settle has no drift reward. | yes | yes | no | full | Presence precision, inflation guard and non-punitive session end all survived. |
| S7 - simulation-runs-server-side | 4 | included | RECON §System: "Canonical bird life belongs on the server" and clients "never own personality state". | PLAN §§2,5,8,9: server tick, database canonical store, append-only events, no client trait writes, no LWW. | yes | yes | no | full | Server tick, multi-device coherence and no-last-write-wins failure mode all appear. |
| S8 - privacy-first-on-bird-data | 2 | included | RECON §System: "Privacy boundaries are product design" and aggregate-only metrics pipeline has no event payloads. | PLAN §§2,3,8: separate metrics pipeline, encrypted email, UUID-only references, no per-bird/per-account telemetry. | yes | yes | no | full | The privacy boundary is technical and product-level, not merely policy language. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECON §System: accessibility is "an equivalent aviary experience, not a reduced status page". | PLAN §§1,6,7,8: complete alternatives, cross-fade motion, naturalist narration, captions, keyboard, first-slice accessibility review. | yes | yes | no | full | Equivalent charm, designed alternatives and launch-time accessibility all survive. |

Multi-layer system-level whys:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: PLAN §§2,5,6,7,8 show quiet field/no spinner, server tick, mid-action first frame, procedural calls, narration/reduced-motion charm.
- S2: PLAN §§1,4,6,8 show no textual return welcome, visit-notification default refusal, no frequency metrics and no gamification copy.
- S3: PLAN §§1,5,7 show naturalist prose in observations, captions, narration and notebook templates, with names/species/perch detail.
- S4: PLAN §§1,6,7 show two starters/seven cap, one unscrollable scene, sparse top bar and recognizability-focused audio.
- S5: PLAN §§1,4,7 show naturalist product surfaces and matter-of-fact account/error/settings surfaces.
- S6: PLAN §§1,5,8,9 show presence conjunction, drift weighting, settle/no mandatory goodbye and no public engagement counter.
- S7: PLAN §§2,3,5,8,9 show canonical database state, server tick, append-only events, no client trait writes and migration/rollback constraints.
- S8: PLAN §§2,3,8,9 show encrypted email boundaries, pipeline separation, aggregate-only telemetry, export/deletion and privacy audits.
- S9: PLAN §§1,6,7,8 show complete alternatives, reduced-motion design, captions, narration cadence, keyboard/focus and first-slice accessibility review.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **72.6%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECON: presence requires visible/focus/recent input and keeps drift from background tabs. | PLAN §5: conjunction rule, no open-tab inference, drift integrity and server clamping. | no | partial | Recovered precision and inflation risk; silent population-wide failure layer is thin. |
| F2 | drift-function | 4 | yes | included | RECON: main exposure signal with one-week instrument and three-week user targets. | PLAN §5/§8: smoothed exposure, per-day caps, one-week/three-week calibration cases. | no | partial | Calibration survived; Tamagotchi-vs-screensaver failure framing did not. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECON: "never subtract" and a two-week absence should read as quiet, not distress. | PLAN §§1,5,9: no punitive absence, no hunger/death/distress, quiet without reversing personality. | no | full | All non-punitive drift layers are recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | RECON: rejects recorded loops and uses varied rhythm, pitch, pauses and timbre. | PLAN §§1,5,7: procedural motifs, WebAudio synthesis, no recorded loops, silence fallback. | no | partial | The no-loop mechanism survived; the chorus artifact/audible-dead-software rationale is incomplete. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECON: pose and low-rate idle actions make identity and state visible without numbers. | PLAN §§5-6: mood/personality choose preen, scan, shuffle and tilt actions. | no | full | Recovered as mood legibility through motion instead of labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECON: hard cap tied to visibility and recognition; audio validated at two through seven birds. | PLAN §§1,6,7,8: cap, safe bounds, listening tests at two to seven birds. | no | full | The recognizability ceiling is recovered. |
| F7 | vector-persistence | 4 | yes | included | RECON: bird records preserve traits, mood, call signature and migration continuity. | PLAN §§2,3,5,8,9: canonical database, server-only vectors, migrations preserve bird state, no client personality writes. | no | full | Server persistence, identity loss risk and downstream no-LWW model survive. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECON: raw personality numbers stay out of ordinary UI and export is not a stats panel. | PLAN §§2,7: snapshot does not expose raw numbers; narration/captions avoid trait numbers/raw labels. | no | full | Recovered as protection against stat-panel treatment. |
| F9 | return-greeting | 4 | yes | included | RECON: exactly one greeting bird, weighted by boldness/warmth/mood/absence, with procedural behavior. | PLAN §6: one greeting bird, absence-length variation, weighted draw, procedural glance/call and stagger. | no | full | The greeting as noticed behavior rather than generic arrival survived. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECON: no welcome toast or absence tally; bird greeting choreography carries return. | PLAN §§1,6: no textual return welcome; never show welcome toast or absence tally. | no | partial | Recovered the rule and extension to absence tally; not the well-meaning-toast/reframing rationale. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | Rule is present via rejected mandatory goodbye, but the chore/ordinary-close rationale is not recovered. |
| F12 | field-notebook-prose | 4 | yes | included | RECON: sparse naturalist observations, not status strings, trait numbers or visit-frequency feedback. | PLAN §§1,5,8: deterministic reviewed naturalist templates, sparse budget, immutable entries, no generic status strings. | no | full | Naturalist prose, anti-event-log treatment and rarity/read-only constraints survive. |
| F13 | presence-accounting | 4 | yes | included | RECON: focus/visibility/recent input prevents background-tab and simultaneous-tab inflation. | PLAN §5/§8: qualifying intervals, heartbeat expiry, server clamping, all-night-open simulations. | no | partial | Mechanics and inflation risk are present; the silent no-test failure layer is not fully reconstructed. |
| F14 | no-streak-counter | 4 | yes | included | RECON: no participation metric/frequency metric/streaks; acceptance review covers hidden settings and analytics. | PLAN §§1,5,8: no streaks/scores/frequency metrics; reject visit-frequency statements and public engagement counters. | no | full | The counter, intention-rotation and disguised-surface refusal are recovered. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECON: scene starts as aviary, quiet field if needed, deterministic mid-action phase. | PLAN §§2,6,8,9: paint quiet field, enter at simulated phase, first frame mid-action, reduce loading before animation. | no | full | Mid-action load, server snapshot and quiet field/no-spinner rationale all survived. |
| F16 | synthetic-account-id | 4 | yes | included | RECON: synthetic UUIDs; email forbidden in partition keys, logs, traces and analytics. | PLAN §§3,8,9: UUID-only references, encrypted email fields, no email in logs/metrics, synthetic-ID audit. | no | partial | PII leak rationale survived; impossible-to-retrofit layer did not. |
| F17 | server-side-sim-tick | 4 | yes | included | RECON: one-minute tick for every aviary including no connected client; canonical server writer. | PLAN §§2,5,8,9: server tick, clients render snapshots, multi-device convergence, no client-owned personality. | no | full | All server-side simulation layers are recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECON: single server writer, ordered append-only events, two devices converge without LWW vector replacement. | PLAN §§2,5,8,9: additive deltas, clients submit events only, server tick writes vectors, no client mutations. | no | full | The anti-LWW implementation rule and cross-device failure risk are recovered. |
| F19 | sync-conflict-tone | 2 | yes | included | RECON: system/account surfaces should be clear, not narrated as bird behavior. | PLAN §§1,4,7: identity/settings/errors/account flows use direct matter-of-fact copy and clear quiet errors. | no | full | Recovered the system-clarity exception to naturalist voice. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECON: metrics pipeline uses allowlisted aggregates and no per-bird state/payloads. | PLAN §§2,8: operational telemetry only; no per-account engagement dashboards, average bird drift or population analyses. | no | partial | Technical boundary survived; private-relationship-as-not-data-product rationale is only implicit. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECON: visitors observe canonical projection, cannot use owner commands, and cannot affect simulation presence. | PLAN §4: same projection as host, no private notebook/account data, visitor presence never enters ledger. | no | full | Recovered observation-not-co-presence and no accidental drift. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | none | yes | none | RECON explicitly marks "No visit announcement" as NOT RECOVERABLE FROM PLAN. |
| F23 | no-leaderboards | 2 | yes | included | RECON: exclusions preserve a private, non-social, non-gamified aviary. | PLAN §§1,8: reject public discovery, feeds, rankings, profiles and per-account engagement dashboards. | no | full | Recovered as refusal of public comparison/social ranking surfaces. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECON: naturalist narration from the same snapshot/event model, not raw labels. | PLAN §§7-8: running prose, names/species/perch behavior, no trait numbers, user studies and editorial review. | no | full | Equivalent prose narration and implementation guardrails are recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECON: same poses/events with cross-fades; mood, calls, notebook and drift remain intact. | PLAN §§6-8: reduced-motion cross-fades, no leaf drift, slowed palette shifts, accessibility tests from early slices. | no | full | The accessibility alternative remains the aviary rather than a stripped fallback. |
| F26 | ttfb-500ms | 2 | yes | included | RECON: first bird visible within 500ms prevents the first frame appearing as app loading. | PLAN §8: 500 ms over 4G, auth/snapshot included, reduce critical path before loading animation. | no | full | Recovered the affective-performance bridge. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECON: rejects scores/streaks/badges/levels and forbids tuning with scores or notifications. | PLAN §§1,8,9: no gamification copy, no frequency metrics, hidden settings/analytics review, no scores/notifications. | no | partial | The absolute refusal survives; temptation and gradual erosion layers are compressed. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECON: rejects hunger, death, distress and punitive absence; birds become quiet rather than distressed. | PLAN §§1,5,9: no hunger/death/distress, no negative drift, two-week absence reads as quiet. | no | full | Recovered observational, non-custodial relationship. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Starter-bird provisioning is present, but arrival-not-catalog/avatar rationale is absent. |
| F30 | age-based-bird-offers | 4 | yes | included | RECON: aviary age alone controls eligibility because new birds are never gated on engagement. | PLAN §§1,4,5,8: age-only adoption transaction, no engagement gate, timezone rules prevent extra eligibility. | no | partial | Age-not-attention survived; unlock-economy erosion layer is not fully reconstructed. |
| F31 | stable-bird-identity | 4 | yes | included | RECON: stable identity protects birds from regeneration/reset through migration. | PLAN §§1,3,8,9: stable UUIDs, never regenerate, migrations preserve UUID/trait/mood/call signature. | no | partial | Identity continuity and reset avoidance survive; distinction from mere vector persistence is thin. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECON: persisted mood and timer prevent mood reset on page open. | PLAN §5: persist mood/timer across sessions; never reset on page open; daily-ish equilibrium movement. | no | full | Recovered as continuity rather than neutral startup default. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only notebook rule appears, but observer-record-not-journal rationale is absent. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECON: export is an owner-only data portability surface, separate from stats. | PLAN §§2,4: account export includes vectors as owner-only data portability and never a stats panel. | no | full | Recovered the owner-copy/data-portability rationale. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECON: deletion allows 30-day recovery, then hard purges dependent relationship records. | PLAN §§3-4: mark deletion, allow restore in 30 days, hard purge birds/vectors/notebook/telemetry/account-tied records. | no | partial | Hard-delete privacy and complete purge survive; accidental-regret relationship rationale is not explicit. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECON: aggregate-only telemetry rejects account IDs, bird IDs, event payloads and per-bird state. | PLAN §§2,8: separate pipeline, metric allowlists, no per-account engagement dashboards or population drift analyses. | no | full | Recovered the technical observability boundary. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECON: explicitly addressed one-time invitations with active grants and revocation. | PLAN §4: issue specifically addressed invitation, no owner commands, revoke and recheck grants on every snapshot. | no | full | Recovered per-invite deliberate sharing and host control. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | On-demand visit log is present, but transparency-not-notification-surface rationale is absent. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECON: visitor endpoint returns the same canonical bird/day/weather projection as the host. | PLAN §4: visitor sees same projection as host with no owner-only commands or special private data. | no | full | Recovered actual-aviary observation rather than staged visitor mode. |
| F40 | narration-cadence-slow | 4 | yes | included | RECON: one observation every 30-60s and queue coalescing avoid flooding live regions. | PLAN §7: 30-60 second cadence, prompt prioritized observations, coalescing, sustained listening comfort tests. | no | partial | Queue-overwhelm and sparse-event layers survived; matching the visual aviary rhythm is not explicit. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | no | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | yes |
| F10 | yes | no | yes |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | yes | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | no | yes |
| F30 | yes | yes | no |
| F31 | yes | no | yes |
| F35 | no | yes | yes |
| F40 | no | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---|---|
| Possible gold whys | 49 | From gold_why_totals |
| Possible total weight | 152 | Fixed full-instance possible weight |
| Reachable gold whys | 49 | S whys always included; all F anchors captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 115 / 152 | Weighted numerator over included whys |
| Whys with reconstruction evidence | 43 | Rows with non-none rationale evidence from frozen reconstruction |
| Whys with PLAN grounding | 44 | Rows with non-none rationale grounding from PLAN |
| `rule_without_why` cases | 5 | Operational rule survived without the gold rationale |
| `plan_only_not_reconstructed` cases | 1 | S4 was cross-cutting in PLAN but not reconstructed as system intent |
| `ungrounded_reconstruction` cases | 0 | No material ungrounded rationale was credited |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (weighted) | Recovery rate |
|---|---|---|---|
| Functional whys | 54 | 42 | 77.8% |
| Affective whys | 98 | 73 | 74.5% |
| Weight-2 whys | 44 | 33 | 75.0% |
| Weight-3 whys | 108 | 82 | 75.9% |
| System-level whys | 28 | 25 | 89.3% |
| Feature-level whys (reachable) | 124 | 90 | 72.6% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys scored 42/54 (77.8%), slightly above affective whys at 73/98 (74.5%). Functional architecture such as S7, F17, F18, F26 and F36 survived especially well.
- **Weight-3 vs weight-2.** Weight-3 whys scored 82/108 (75.9%), essentially tied with weight-2 whys at 33/44 (75.0%). High-weight layering did not guarantee full recovery.
- **System-level vs feature-level.** The headline split is system 89.3% versus feature 72.6%. The plan preserved philosophy strongly, but the reconstruction often compressed per-feature exceptions into mechanisms.
- **Multi-layer recovery.** Dropped layers were usually downstream consequences: F2 lost the Tamagotchi/screensaver failure band, F10 lost the well-meaning-toast reframing, F30 lost the unlock-economy erosion, and F40 lost the visual-rhythm layer.
- **Subdomain patterns.** Accounts/sync and accessibility were strongest; social optional and targeted headroom whys leaked more, especially F22, F37/F38 and F39-style social-affective distinctions.
- **Evidence-bound effects.** F11, F22, F29, F33 and F38 were captured features with rules present, but v06 denied recovery because the frozen reconstruction did not recover the gold rationale. S4 became a plan-only partial because restraint was cross-cutting in the plan but not elevated in system-level reconstruction.

What the failure shape suggests: the plan is unusually complete as an implementation artifact, but plan-derived reconstruction still tends to retain architectural invariants better than product-reasoning exceptions.

## 4. Recommendations for v2 hardening

- Preserve targeted headroom whys like F29, F33 and F38. They remained discriminative even when planning coverage was near-saturated.
- Add more paired social/privacy whys that distinguish privacy-as-control from privacy-as-relationship. This run often collapsed those into generic access-control language.
- Clarify the system-level cross-cutting rule for cases where B reconstructs several feature-level instances but does not name the system principle. S4 was the main subjective call.
- Keep the evidence-bound operator. It separated rule retention from why retention without penalizing honest non-recovery.

## 5. Methodology caveats

- **Fresh-context fidelity.** Validity audit passed: no gold IDs, no scorer vocabulary and headings follow the required two-section reconstruction shape.
- **Single-run-at-temperature limitation.** This is one run only, with no variance signal.
- **Borderline capture calls.** Feature 58 was counted captured despite incomplete night-state detail. Features 7, 69 and 116 were missed; none are F1-F40 anchors.
- **System-level cross-cutting.** S4 was the subjective system-level call: the plan had multiple restraint inheritances, but the reconstruction did not name restraint as a system principle.
- **Confabulation cases.** No material ungrounded reconstruction was credited.
- **Evidence-bound denials.** The most important denials were F11, F22, F29, F33 and F38.
- **Rule-without-why cases.** Five included whys preserved rules without recoverable rationale.
- **Operational compromise.** TIMING.json contained phase1 and phase2a only; phase2b timing was unavailable in-context and omitted.

End of report.
