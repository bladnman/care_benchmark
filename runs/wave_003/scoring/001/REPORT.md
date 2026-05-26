# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Denominator inclusion is separate from recovery; every S1-S9 and F1-F40 row cites frozen reconstruction evidence and PLAN grounding.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **93.3%** |
| Intent fidelity | **61.0%** |
| Combined quality | **9294** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **55.9%**

**(Planning, fidelity) coordinate:** `(93.3, 61.0)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-26T22:16:00Z |
| Candidate model | qwen3.7-max |
| Candidate effort | unknown |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **112 / 120** = **93.3%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 17 | 85.0% |
| aviary_layout.md | 18 | 17 | 94.4% |
| accounts_sync.md | 18 | 16 | 88.9% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **112** | **93.3%** |

Per-feature detail:

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | captured |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Covered by feels-alive risk, first-frame conceit, variation, and procedural calls. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured operationally through no engagement counters, no push/ping/email, and no announcement framing. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | captured |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | captured |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | captured |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No explicit glossary or equivalent term reference surface. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | captured |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | captured |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | captured |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | captured |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | captured |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | captured |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | captured |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | captured |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | captured |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | captured |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | captured |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | captured |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | captured |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | captured |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | captured |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | captured |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | captured |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | captured |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | captured |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | captured |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | captured |
| 29 | Mood persistence across sessions | bird_engine.md | yes | captured |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | captured |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | captured |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | captured |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | captured |
| 34 | Greeting variation by absence length | interactions.md | yes | captured |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | captured |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | captured |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit textual welcome/toast/banner prohibition. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | captured |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | captured |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | captured |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | captured |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | captured |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | captured |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | Settle exists, but closing-without-settle equivalence is not specified. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | captured |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | captured |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by server-generated notebook entries and read-only GET API surface. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | captured |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | captured |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | captured |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Server continuation is present, but client render pause while hidden is not specified. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | captured |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | captured |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | captured |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | captured |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | captured |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | captured |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | captured |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | captured |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | captured |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | captured |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | captured |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | captured |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | captured |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | captured |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | captured |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | captured |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state is described; the plan assumes starter birds are present. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured through day/night color phases and naturalist palette choices, though accent-color refusal is compressed. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | captured |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | captured |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | Magic-link expiry is not specified. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | captured |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | captured |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | captured |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | captured |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | captured |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | captured |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | captured |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | captured |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | captured |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | captured |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | captured |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | captured |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | captured |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | captured |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | captured |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | captured |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | captured |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | captured |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | captured |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | captured |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | captured |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | captured |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | captured |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | captured |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | captured |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | captured |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | captured |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | captured |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | captured |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | captured |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | captured |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | captured |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | captured |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | captured |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | captured |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | captured |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | captured |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | captured |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | captured |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | captured |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | captured |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | captured |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | No explicit browser support matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | captured |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | captured |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | captured |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | captured |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md: feel alive through continuity, variation, subtlety; calls never repeat identically; first-frame conceit must be flawless | PLAN.md: quiet field/no spinner; procedural synthesis; varied greetings; non-looping idle motion; non-repeating calls | yes | yes | no | full | Aliveness is recovered as a cross-cutting concern. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md: not an engagement loop; social/growth surfaces stay quiet and opt-in; no announcement framing | PLAN.md: no gamification/no push; gentle notebook offers; no announcement framing; visit notifications default quiet | yes | yes | no | partial | Quietness survives; processed-vs-seen and toast-slip rationale do not. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md: naturalist voice is specific to the bird and moment with no gamification/announcement framing | PLAN.md Appendix C: lowercase, present-tense, specific, bird-centered voice; curated notebook templates | yes | yes | no | full | Specificity-over-generic-system-voice survives. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md: single horizontal scene NOT RECOVERABLE; cap 7 tied only to age growth | PLAN.md: two-bird start, seven-bird cap, single scene, no custom scenes, restrained top-bar UI | no | yes | yes | partial | Plan preserves restraint; reconstruction misses the rationale. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md: product surfaces use naturalist voice; sign-in/errors/settings/sync use matter-of-fact voice | PLAN.md Appendix C plus error/reconnection and narration/caption surfaces | yes | yes | no | full | Voice split and exception are explicit. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md: presence pings aggregate into drift; instrumentation detects presence inflation; relationship is non-punitive | PLAN.md: visibility + focus + activity presence detection; presence drives drift; no streaks/Tamagotchi | yes | yes | no | partial | Presence/drift and inflation survive; settle/tab-close equivalence is missing. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md: client is renderer/event emitter; server is sole state authority; one canonical record and no LWW personality arbitration | PLAN.md: client/server split; tick loop; canonical record; single-writer conflict prevention | yes | yes | no | full | Server authority, sync coherence, and conflict prevention are recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md: synthetic UUIDs, email never key/log field, aggregate-only observability, no per-bird/per-account telemetry | PLAN.md: email never key/partition/log field; observability excludes relationship data; telemetry pipeline separated | yes | yes | no | full | Technical privacy boundary survives strongly. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md: accessibility is core, not late add-on; reduced motion cannot be partially implemented | PLAN.md: narration, reduced motion, captions, keyboard, contrast; accessibility critical path from day one | yes | yes | no | full | Charm, completeness, and launch-critical accessibility survive. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

Cross-cutting evidence appendix:

- S1: return-greeting variation, procedural calls, idle motion, quiet loading, reduced motion, first-frame risk.
- S2: no gamification, no push/ping/email, quiet visit notifications, gentle offers, no announcement framing.
- S3: naturalist notebook/narration/captions, bird naming, hidden raw traits, no public comparison surfaces.
- S4: two-bird start, seven-bird cap, single horizontal scene, no custom scenes, restrained top-bar UI.
- S5: naturalist product surfaces; matter-of-fact errors, sign-in, sync, account, and accessibility settings.
- S6: visibility/focus/activity presence detection, presence-driven drift, no streaks/Tamagotchi, inflation checks.
- S7: server tick, canonical record, client snapshots, no LWW personality state, append-only event log.
- S8: synthetic UUID, encrypted email, aggregate telemetry, separated telemetry pipeline, export/deletion controls.
- S9: screen-reader prose, reduced motion, captions, keyboard navigation, focus indicators, accessibility critical path.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **55.9%**.

Reachable feature-level whys: **38 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: presence pings aggregate into drift; valid presence is flushed; instrumentation checks presence inflation | PLAN.md: presence detection visibility + focus + activity; PresenceSession fields; drift uses presence_time | no | partial | Presence feeds drift, but three-signal precision and silent-corruption rationale are compressed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: low-pass drift accumulates slowly; base rate gives visible movement after about three weeks | PLAN.md §5.2 low-pass filter; calibration table/tests; §12.1 too-fast/too-slow risk | no | partial | Low-pass and calibration survive; Tamagotchi/screensaver consequence is absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: neglect produces ambient quietness; clamping makes traits never decrease | PLAN.md: no Tamagotchi; non-negative clamping; zero-presence test no trait decrease | no | partial | Non-punitive monotonicity survives; two-week return consequence is absent. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: procedural calls vary by species/mood/context without repeating or downloading audio | PLAN.md §8.2 motif variation; §8.6 no audio files; calls never repeat identically | no | partial | No-loop aliveness and audio cascade survive; chorus phase rationale is absent. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md: mood-to-idle mapping grounds mood in visible behavior | PLAN.md §7.4 mood table maps states to idle motion | no | full | Visible behavior carries mood without a label. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md: 2 starter birds with cap 7 tied to age-based growth | PLAN.md: max seven; bird ramp reaches seventh bird at 450 days | yes | none | Cap is a rule, but empirical recognizability ceiling is absent. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md: drift audit trail and running sum; corruption means the bird the user knows is gone | PLAN.md: server mutates personality; DriftRecord audit trail; only simulation tick writes personality | no | full | Canonical storage, relationship loss, and sync consequences survive. |
| F8 | personality-vector-never-numerical | 2 | yes | included | RECONSTRUCTION.md: render_hints/call_hint expose derived behavior while client never knows underlying numbers | PLAN.md: render_hints derived from personality but never expose raw trait values | yes | none | No-number rule survives, not stat-management rationale. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md: greeting uses absence, mood, boldness, social warmth and must be genuinely varied | PLAN.md §7.5 selects one greeting bird and varies greeting by absence/mood/personality | no | partial | Selection and variation survive; notice-never-announce consequence is absent. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor missed: no explicit welcome toast/banner/modal prohibition. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor missed: no close-tab equivalence. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECONSTRUCTION.md: notebook is naturalist observation, not event logging; entries are rare | PLAN.md §5.6 curated naturalist templates; rarity probability; Appendix C voice | no | partial | Naturalist prose and event-log refusal survive; read-only/not-feed distinction is incomplete. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: valid presence flushed every 30 seconds and aggregates into drift | PLAN.md: visibility + focus + activity; PresenceSession fields; drift uses presence_time | no | partial | Presence-to-drift survives, not per-signal failure cases. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: no achievements, streaks, scores, badges, green-dot calendars, XP, ranks, tiers, or engagement counter | PLAN.md: absolute gamification non-goal; no engagement counters; notebook avoids user behavior observations | no | partial | Refusal and adjacent disguises survive; managing-number rationale is mostly absent. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: immediate Canvas first frame; slow path quiet field with no spinner | PLAN.md: inlined initial state; quiet field/no spinner; first-frame conceit must be flawless | no | full | Already-running frame, snapshot implementation, and no-spinner fallback survive. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: synthetic UUID never derived from email; email never key/partition/log field | PLAN.md: synthetic UUID; email never key/partition/log field | no | partial | Identifier and PII leakage concern survive; retrofit consequence is absent. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md: tick runs whether or not client connected; all clients read canonical result | PLAN.md: tick loop; one canonical record; client never owns personality/mood state | no | full | Server tick, sync coherence, and avoided client divergence survive. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md: no CRDT/OT/LWW on personality; clients submit events and never own state | PLAN.md: no LWW arbitration; append-only/idempotent events; only tick writes personality | no | partial | Implementation rule survives; invisible lost-drift example is absent. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | RECONSTRUCTION.md: sync conflicts use matter-of-fact voice with no warmth pretending to be useful | PLAN.md Appendix C matter-of-fact sync conflicts; reconnection error surface | no | full | Error-context voice exception survives. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: observability excludes per-bird/per-account data; separate DB/allowlists/tests protect boundary | PLAN.md: not-measured telemetry list; separate analytics pipeline and allowlist | no | partial | Rule and pipeline boundary survive; relationship-as-data-product rationale is absent. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visitor snapshots exclude interaction capabilities; ambient visitor view not shared control | PLAN.md: read-only ambient visitor view/no co-presence; filtered visitor snapshot | no | full | Observation-not-co-presence survives. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: social/growth surfaces stay quiet and opt-in; no push/ping/email | PLAN.md: no push/ping/email; visit notification setting default false; visits default off | yes | none | No-notification mechanism survives, not attention-driver rationale. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | RECONSTRUCTION.md: no profiles/follows/feeds/leaderboards/show-off mode; no leaderboard/streak metrics computed | PLAN.md: no leaderboards/discovery/public feeds; no underlying metrics computed | yes | none | Refusal survives, not comparison-product-shift rationale. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: ARIA live region gives naturalist descriptions of idle state and user events | PLAN.md §9.1 naturalist narration templates and voice continuity | no | partial | Running prose and same-product voice survive; ARIA-automation warning is absent. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md: reduced motion is cross-fade surface; audio/captions/narration/notebook/interactions remain | PLAN.md §9.2 cross-fades/static overlays while core audio/captions/narration/notebook remain | no | partial | Different rendering and retained core survive; stripped-fallback consequence is absent. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md: inlined state and Canvas first frame target 450ms; performance protects the illusion | PLAN.md §10.2 450ms budget under 500ms; fast path avoids visible loading | no | full | Performance-as-felt-aliveness bridge survives. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION.md: not an engagement loop; gamification refusal is load-bearing; metrics are not computed | PLAN.md: absolute gamification non-goal; review checklist; no computed metrics | no | full | Rule, temptation, and creep-prevention rationale survive. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md: birds do not die/get hungry/show distress; neglect produces ambient quietness | PLAN.md: no Tamagotchi; monotonic drift; zero presence never decreases traits | no | full | Non-custodial, non-punitive relationship survives. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md: System-selected starter birds: NOT RECOVERABLE FROM PLAN | PLAN.md: system-selected two starter birds and user-assigned names | yes | none | Adoption rule exists, but animals-arrived-not-catalog rationale is absent. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md: new bird offers are gradual, non-intrusive growth over months/year via notebook messages | PLAN.md: age-based bird growth; bird ramp by aviary age; gentle notebook offer/no expiry | no | partial | Growth-over-time survives; reward/economy danger does not. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md: bird IDs never change; corruption is catastrophic because the bird the user knows is gone | PLAN.md: stable Bird.id; corruption impact is catastrophic | no | partial | Same-individual continuity and retroactive loss survive; vector distinction is absent. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md: mood state machine retains current mood if no strong signal; snapshots carry canonical mood | PLAN.md: Bird.current_mood stored; canonical state per account; pull after hidden/focus | yes | none | Persistence mechanism is present, but no-neutral-reset rationale is absent. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md: field notebook is naturalist observation, not event logging; endpoint not recoverable | PLAN.md: only GET notebook endpoint; server generates entries; no edit/delete API | yes | none | Read-only is inferable, but observer-record-vs-journal rationale is absent. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md: Account export: NOT RECOVERABLE FROM PLAN; Account Service owns exports | PLAN.md: account export JSON; POST /account/export; Account Service export generation | yes | none | Export is an endpoint, not the user relationship-copy rationale. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md: 30-day soft-delete window allows account recovery before hard deletion | PLAN.md: deleted_at/hard_delete_scheduled_at; delete/recover endpoints | no | partial | Accidental-regret grace survives; privacy/residue layers are absent. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: aggregate-only metrics exclude per-account and per-bird data to preserve privacy | PLAN.md: aggregate metrics and not-measured list; telemetry field allowlist | no | full | Technical observability boundary survives. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md: invitations are off by default, per-invite opt-in, revocable, expire after 30 days | PLAN.md: per-invite/revocable visits; no friend-of-friend or implicit social surfaces | no | full | Deliberate named access survives. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md: Social Service maintains visit log; Visit log: NOT RECOVERABLE FROM PLAN | PLAN.md: visit log; GET /social/visits; VisitLogEntry model | yes | none | Log exists, but transparency-without-notification rationale is absent. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md: social non-goals prevent show-off mode | PLAN.md: no show-off mode; visitor endpoint read-only snapshot | yes | none | No show-off mode survives, but real-birds-not-marketing rationale is absent. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: narration every 30-60 seconds; user events preempt; queue capped/replaced to prevent stacking | PLAN.md §9.1 30-60s cadence, priority events, capped queue, naturalist prose | no | full | Slow cadence, queue safety, and observational sparseness survive. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | no |
| F2 | yes | yes | no |
| F3 | yes | yes | no |
| F4 | yes | no | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | no |
| F10 | no | no | no |
| F12 | yes | yes | no |
| F13 | yes | no | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | no |
| F27 | yes | yes | yes |
| F30 | yes | no | no |
| F31 | yes | no | yes |
| F35 | yes | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 47 | 9 system + 38 feature whys |
| Excluded unreachable feature whys | 2 | F10 and F11 anchors missed |
| Recovered / reachable weight | 89.0 / 146 | Weighted recovery over included whys |
| Whys with reconstruction evidence | 47 | Evidence fields cite reconstruction text, including negative evidence |
| Whys with PLAN grounding | 47 | Included whys had plan grounding |
| `rule_without_why` cases | 11 | Mechanism survived without gold rationale |
| `plan_only_not_reconstructed` cases | 0 | Counted at whole-why level |
| `ungrounded_reconstruction` cases | 0 | No clear ungrounded whole-why recoveries |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (full + partial-weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 34.0 | 63.0% |
| Affective whys | 92 | 55.0 | 59.8% |
| Weight-2 whys | 42 | 21.0 | 50.0% |
| Weight-3 whys | 104 | 68.0 | 65.4% |
| System-level whys | 28 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 118 | 66.0 | 55.9% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional recovery was 34/54 (63.0%); affective recovery was 55/92 (59.8%). Functional infrastructure was clearer, while relationship-protecting social exceptions leaked rationale.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 68/104 (65.4%), better than weight-2 at 21/42 (50.0%). High-weight mechanics repeated across architecture, risks, and rollout.
- **System-level vs feature-level.** System-level recovery was much stronger: 82.1% vs 55.9%. Philosophy survived broadly; feature exceptions leaked first.
- **Multi-layer recovery patterns.** Layer 1 usually survived. Layer 2/3 consequences were common losses: F2 lost the Tamagotchi/screensaver failure band, F9 lost notice-never-announce consequence, and F30 lost reward-economy erosion.
- **Subdomain patterns.** Accounts/sync and accessibility were comparatively strong. Social optional and relationship-boundary features were weaker: F22, F23, F38, and F39 mostly retained prohibitions without why.
- **Evidence-bound effects.** F6, F8, F22, F23, F29, F32, F33, F34, F38, and F39 look superficially covered in a rule-only review, but v06 correctly denies why recovery.

## 4. Recommendations for v2 hardening

- Keep targeted feature-level exception whys; they created real headroom even for a high-coverage plan.
- Add more tests around social/privacy/account surfaces where implementation rules are easy to list and rationales are easy to compress away.
- Preserve multi-layer scoring and add a rollup for which layer class drops most often; here, downstream consequences leaked most.
- Keep the system-level cross-cutting bar at three inherited decisions; S4 shows it separates preserved structure from recovered rationale.
- Continue requiring exact reconstruction evidence and PLAN grounding to prevent mechanism-only rows from inflating fidelity.

## 5. Methodology caveats

- **Fresh-context fidelity.** Based on the supplied setup, phase 2A was frozen and separate. I did not modify `RECONSTRUCTION.md`.
- **Single-run-at-temperature limitation.** This is one sample; there is no variance signal inside this score.
- **Borderline capture calls.** I leaned inclusive on color palette, field-notebook read-only, mood persistence, and bird-chosen perch because the plan gave enough implementation direction. Misses were reserved for explicit omissions.
- **System-level cross-cutting.** The most subjective calls were S2, S4, and S6. Their partial scores reflect preserved plan structure but incomplete reconstructed rationale.
- **Confabulation cases.** I found no clear whole-why recovery ungrounded in the plan. Some rows gave non-gold rationales, treated as rule-without-why.
- **Evidence-bound denials.** The main denials were rule-without-why, not lack of plan grounding.
- **Operational compromise.** `TIMING.json` supplied phase 1 and phase 2A only for run 001; the score JSON includes only those bounded timing fields.

---

End of report.
