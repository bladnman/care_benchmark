# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Reconstruction was frozen; scoring uses exact reconstruction evidence and PLAN grounding.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **99.2%** |
| Intent fidelity | **70.4%** |
| Combined quality | **9887** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **67.7%**
- (Planning, fidelity) coordinate: `(99.2, 70.4)`

No low-confidence banner applies.

### Run metadata

| Field | Value |
|---|---|
| Run number | 1 |
| Run label |  |
| Timestamp | 2026-09-24T08:15:22Z |
| Candidate model | gpt-6-sol |
| Candidate effort | high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **119 / 120** = **99.2%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **119** | **99.2%** |

Per-feature detail:

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes |  |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes |  |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes |  |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes |  |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes |  |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes |  |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline captured: terms are defined across plan, not as a glossary. |
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
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes |  |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes |  |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes |  |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes |  |
| 29 | Mood persistence across sessions | bird_engine.md | yes |  |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes |  |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes |  |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes |  |
| 33 | Return-greeting on viewer arrival | interactions.md | yes |  |
| 34 | Greeting variation by absence length | interactions.md | yes |  |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes |  |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes |  |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes |  |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes |  |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes |  |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes |  |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes |  |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes |  |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes |  |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes |  |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes |  |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes |  |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes |  |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes |  |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes |  |
| 50 | No streak counter, no "days visited" display | interactions.md | yes |  |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes |  |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes |  |
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
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes |  |
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
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Missed: no explicit privacy-policy link in account settings. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes |  |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes |  |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Borderline captured: explicit invite-only visits imply default off. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes |  |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes |  |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Borderline captured: comments/profiles/discovery are excluded; chat/avatar is implicit. |
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

System-level fidelity: **82.1%** (23.0/28.0).

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | R System: "ongoing aviary", "no spinner", procedural non-canned calls | PLAN §§1,5,6,8: ongoing first frame, varied greetings, quiet field, procedural calls | yes | yes | no | partial | Aliveness/procedural life survived; downstream staleness consequence did not. |
| S2 - notice-never-announce | 4 | included | R System: "not an engagement game", "no textual return greeting", no badge/default notification | PLAN §§1,4,5,9: no textual greeting, no streaks, no visit badge, no push engagement | yes | yes | no | partial | Announcement refusals survived; processed-vs-seen rationale mostly absent. |
| S3 - charm-from-specificity | 2 | included | R System: "lowercase, specific naturalist observation"; notebook/narration naturalist prose | PLAN §§1,5,7: specific naturalist voice, bird-specific notebook, no raw status/trait narration | yes | yes | no | full | Specific naturalist charm is recovered. |
| S4 - restraint-over-richness | 2 | included | none | PLAN §§1,6,9: two-to-seven birds, one screen, no panning/chrome, recognizability and density gates | no | yes | yes | partial | PLAN preserves restraint, but B did not identify the depth-over-variety principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | R System: naturalist aviary voice; matter-of-fact auth/account/errors/settings | PLAN §§1,4,7: same voice split across product and system surfaces | yes | yes | no | full | Voice split recovered. |
| S6 - presence-is-real-interaction | 4 | included | R System: "Honest presence without punishment"; all three signals; settle/tab-close same | PLAN §5: visibility + focus + input, presence dominates drift, no penalty for settle/tab-close | yes | yes | no | full | Precise attention signal and no-punishment model recovered. |
| S7 - simulation-runs-server-side | 4 | included | R System: "Server-authored, durable, canonical life"; sole writer; no last-write-wins | PLAN §§2,5,8,10: server tick, client snapshots, additive deltas, no client writes | yes | yes | no | full | Server-side canonical model recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | R System: "Privacy by separation and omission"; analytics restricted from bird/event/notebook state | PLAN §§2,8: analytics boundary and aggregate-only metrics | yes | yes | no | full | Pipeline-level privacy recovered. |
| S9 - accessibility-as-first-class-surface | 4 | included | R System: "designed mode, not a dump of internals"; reduced motion; narration from shared events | PLAN §§6,7,9: designed renderer, naturalist narration/captions, keyboard/focus gates | yes | yes | no | full | Accessible aviary as first-class experience recovered. |

Multi-layer system whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | yes | no | no |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: return greeting variation; procedural calls; mid-action first frame; quiet loading field; reduced-motion/narration charm.
- S2: no textual greeting; no streaks; no visit badge; no push engagement; no engagement leaderboard monitoring.
- S3: naturalist notebook/narration/captions; named birds; hidden numeric traits; no comparison surfaces.
- S4: two-to-seven birds; one screen/no panning; no scene chrome; seven-bird recognizability gates.
- S5: naturalist product copy; matter-of-fact auth/errors/settings/revocation/network status.
- S6: presence conjunction; drift input; settle/tab-close equivalence; no streaks; monotonic no-punishment drift.
- S7: server tick; snapshots; no client personality writes; no last-write-wins; edge snapshot reconciliation.
- S8: synthetic IDs; analytics restrictions; aggregate telemetry; export/deletion controls; no visitor signals.
- S9: reduced-motion renderer; naturalist narration/captions; keyboard/focus; accessibility gates before release.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **67.7%** (84.0/124.0).
Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | R Presence: all three signals; qualified presence only; no retroactive credit | PLAN §5: all-three conjunction; presence dominates drift; visible tab alone not enough | no | partial | Precision and shortcuts recovered; silent population corruption thin. |
| F2 | drift-function | 4 | yes | included | R Drift: slow additive drift; week-one/week-three fixtures; no visible jumps | PLAN §5: low-pass step; measurable week one; visible week three | no | partial | Calibration recovered; failure endpoints lost. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | R System/Drift: traits never decline; never reduce on neglect; healthy familiar birds after absence | PLAN §5/§10: nonnegative deltas; two-week absence; no trait reduction | no | full | No-punishment monotonic drift recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | R Calls: procedural, recognizable, non-canned; no loops/fallback/layered recordings | PLAN §6/§10: versioned grammar, WebAudio, generated chorus, no recorded fallback | no | full | Procedural audio rationale recovered. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Rule appears, but not mood-readable-without-label rationale. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | PLAN §9: chorus recognizability and scene-density tests; seven cap from day one | yes | none | B marked cap NOT RECOVERABLE; plan-only evidence cannot score. |
| F7 | personality-vector-persistence | 4 | yes | included | R System: preserve IDs/vectors; reject regenerated or reset vectors; no client personality writes | PLAN §§2,3,5,10: persisted vectors, server writer, migration invariants | no | partial | Persistence and authority recovered; affective loss layer absent. |
| F8 | personality-vector-never-numerical | 2 | yes | included | R Product: trait numbers have no in-product UI; export not stats surface | PLAN §1: raw vectors are data portability, not an in-product stats surface | yes | none | Partial stats-surface rationale, but not enough for single-layer credit. |
| F9 | return-greeting | 4 | yes | included | R Greeting: one bird notices; absence/boldness/mood; quick glance vs days-away orientation; no fixed list | PLAN §§1,5: one eligible bird, absence/boldness/warmth/mood, seeded variation | no | full | Specific notice-not-copy greeting recovered. |
| F10 | no-welcome-back-toast | 4 | yes | included | R Product: no textual return greeting; bird behavior rather than system copy | PLAN §1: returning host noticed by bird; no textual return greeting | no | partial | Core no-text welcome survived; temptation/downstream layers absent. |
| F11 | settle-is-opt-in | 2 | yes | included | R Presence: settle and tab-close same absence treatment; no penalty | PLAN §5: tab-close and settle yield same absence treatment; no engine penalty | no | full | Optional/no-penalty rationale recovered. |
| F12 | field-notebook-auto-entries | 4 | yes | included | R Notebook: salient observations, rare budget, naturalist prose, not every session/raw event | PLAN §§5,7: rare bird-specific present-tense prose; no attendance/vector/score prose | no | full | Notebook voice, rarity, and observer posture recovered. |
| F13 | presence-accounting | 4 | yes | included | R Presence: all three signals; no retroactive credit; visible tab alone not enough; union devices | PLAN §5: simultaneous conditions; heartbeat lease; union host intervals | no | partial | Implementation precision recovered; silent scale failure compressed. |
| F14 | no-streak-counter | 4 | yes | included | R System/Product: not engagement game; excludes streaks; notebook never reports attendance | PLAN §§1,5: no streaks/visit displays; no attendance prose | yes | partial | No-visit-frequency line survived; number-management rationale partial. |
| F15 | scene-loads-with-motion | 4 | yes | included | R Visual: continuous first scene, mid-action first frame, edge snapshots, quiet field instead of spinner | PLAN §§2,6,8: signed snapshot, pose from phase/server time, no spinner, <500 ms | no | full | Continuing-without-viewer first frame recovered. |
| F16 | synthetic-account-id | 4 | yes | included | R Delivery/Privacy: synthetic UUIDs for joins/logs/messages/metrics; email encrypted on account | PLAN §§2,3,8: synthetic UUID, encrypted email, logs omit emails | no | partial | PII/log boundary recovered; retrofit consequence absent. |
| F17 | server-side-simulation-tick | 4 | yes | included | R Tick: one-minute durable tick for every aviary; server sole writer; clients render snapshots | PLAN §§2,5: durable tick, no connected client required, client snapshots | no | full | Server tick rationale recovered. |
| F18 | no-last-write-wins-personality | 4 | yes | included | R Risk/System: server-only additive deltas; no last-write-wins; clients submit events only | PLAN §§4,5,10: no vector writes, server-authored deltas, no client personality writes | no | full | Anti-LWW rationale recovered. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | none | none | yes | none | Matter-of-fact copy appears, but not evasive-error rationale. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | R Privacy/Ops: analytics restricted; no population drift dashboard or warehouse event-table copy | PLAN §§2,8: analytics blocked from relationship tables; no event-table warehouse | no | partial | Storage/pipeline boundary recovered; relationship-as-data-product layer absent. |
| F21 | visit-read-only-ambient | 2 | yes | included | R Visits: narrow snapshot route, no visitor presence rows/event route/drift | PLAN §§1,4,5,9: visitor token only snapshot; collector suppressed; no visitor drift | no | full | Observation not co-presence recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | R API: visit log only on demand, no badge/default notification; avoids engagement/social pressure | PLAN §4: on-demand log; no badge/default notification; deliberate opt-in only | no | full | Attention-driver refusal recovered. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | none | none | yes | none | Rule listed, but comparison-changes-relationship rationale absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | R Accessibility: narration from snapshot/grammar events, slow naturalist prose, no raw status/perch/trait values | PLAN §7: naturalist prose, not raw codes/perch/trait values, events as observations | no | full | Naturalist anti-state-list narration recovered. |
| F25 | reduced-motion-mode | 4 | yes | included | R Visual: designed second renderer, same state/actions, cross-fades instead of motion | PLAN §6: same state/actions, still-pose/perch cross-fades, no drifting leaves | no | partial | Alternate rendering recovered; cost-of-accessibility consequence absent. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | R Performance: first bird within 500 ms; first usable frame ongoing; performance gate | PLAN §§6,8: first bird <500 ms; first-scene critical path; launch blocker | no | full | Affective-performance threshold recovered. |
| F27 | no-gamification | 4 | yes | included | R System: ongoing aviary, not engagement game; excludes scores/streaks/achievements/levels/leaderboards | PLAN §§1,9: explicit no gamification; no engagement leaderboards | yes | partial | Product-identity refusal survived; temptation/foothold layers absent. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | R Drift/System: no care meters/hunger/distress/death; no penalty; healthy birds after absence | PLAN §§1,5,10: exclude care/hunger/distress/death; never reduce traits; two-week absence fixture | no | full | Observational/no-punishment rationale recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | B marked starter-bird rationale NOT RECOVERABLE. |
| F30 | age-based-new-bird-offers | 4 | yes | included | R Product/Simulation: age-based only, never session count/interaction volume, not attention reward | PLAN §§1,5: age-paced arrivals; offers from creation time only | no | partial | Age-not-attention rationale recovered; economy erosion absent. |
| F31 | stable-bird-identity | 4 | yes | included | R System/Data: identity survives projection/export/rename/visit/migration; never replace IDs | PLAN §§2,3,10: preserve IDs/vectors; stable UUID; immutable IDs | no | partial | Identity continuity recovered; retroactive relationship loss absent. |
| F32 | mood-persists-across-sessions | 2 | yes | included | R Tick: gradual daily-ish reset; no neutral reset at midnight or page open; continuity | PLAN §5: persisted mood; gradual time-of-day pull; never neutral reset | no | full | Mood-continuity rationale recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only survived as rule, but not not-a-user-journal rationale. |
| F34 | account-export-relationship-copy | 2 | yes | included | R Product/Data: export as data portability; vectors behind settings; not stats surface | PLAN §§1,3,4: account JSON export via short-lived verified-account link | no | full | Data portability/user-copy rationale recovered. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | R Product/Data: immediate mark, 30-day recovery, then hard-delete records/artifacts/telemetry | PLAN §§3,4,9: mark, suspend, recover 30 days, hard delete dependent records | no | full | Grace and hard-delete rationale recovered. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | R Ops: aggregate-only metrics; no per-bird/notebook content; no event-table warehouse | PLAN §§2,8: aggregate ops metrics; no per-bird state/presence/notebook prose | no | full | Technical telemetry boundary recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | R Visits: named invitations, scoped viewer sessions, no public profiles/discovery | PLAN §§1,4: named invites and revocation; no public profiles/discovery | no | full | Deliberate named sharing recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | R API: visit log only on demand, no badge/default notification | PLAN §4: host sees visit info on demand; no badge/default notification | no | full | On-demand transparency recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Same projection rule appears, but no show-off/real-witness rationale. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | R Accessibility: 30-60 sec pacing, coalescing, queued user events, avoids flooding queue | PLAN §7: slow prose every 30-60 sec; user events queued; coalesce stale updates | no | full | Cadence and queue-protection recovered. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | yes |
| F10 | yes | no | no |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | no |
| F27 | yes | no | no |
| F30 | yes | yes | no |
| F31 | yes | yes | no |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From constants |
| Possible total weight | 152 | From constants |
| Reachable gold whys | 49 | S whys plus reachable F whys |
| Excluded unreachable feature whys | 0 | Denominator exclusions |
| Recovered / reachable weight | 107.0 / 152.0 | Weighted numerator/denominator |
| Whys with reconstruction evidence | 41 | Exact evidence in frozen reconstruction |
| Whys with PLAN grounding | 43 | Exact plan grounding |
| `rule_without_why` cases | 11 | Mechanism survived without enough rationale |
| `plan_only_not_reconstructed` cases | 2 | S4 and F6 |
| `ungrounded_reconstruction` cases | 0 | None found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 40.0 | 74.1% |
| Affective whys | 98.0 | 67.0 | 68.4% |
| Weight-2 whys | 44.0 | 27.0 | 61.4% |
| Weight-3 whys | 108.0 | 80.0 | 74.1% |
| System-level whys | 28.0 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 124.0 | 84.0 | 67.7% |

---

## 3. Diagnostic patterns

- **Affective vs functional:** Functional whys recovered 40.0/54.0, while affective whys recovered 67.0/98.0. The engine/sync/privacy spine survived better than relationship-shaping affective exceptions.
- **Weight-3 vs weight-2:** Weight-3 whys recovered 80.0/108.0; weight-2 whys recovered 27.0/44.0. Richer implementation scaffolding often gave multi-layer rows partial credit even when downstream consequences leaked.
- **System-level vs feature-level:** System fidelity (82.1%) exceeded feature fidelity (67.7%). The planner preserved philosophy broadly, but feature-specific why text was compressed.
- **Multi-layer recovery:** Layer 3 leaked most often: S1/S2 and F1/F2/F16/F20/F25/F30/F31 are the main examples.
- **Subdomain patterns:** Server simulation, sync, deletion, telemetry, visits, and screen-reader narration were strong. Numeric traits, leaderboards, starter birds, notebook editability, and visitor show-off mode were weak.
- **Evidence-bound effects:** V06 denied credit where the rule existed without the why, especially F5, F19, F23, F29, F33, and F39.

## 4. Recommendations for v2 hardening

- Keep F29-F40; they exposed meaningful rationale loss in a near-complete plan.
- Add more worked examples for single-layer affective whys with multiple clauses, such as F8 and F33.
- Preserve evidence-bound scoring; it separated feature capture from intent recovery cleanly here.
- Clarify system-level plan-only recovery for cases like S4, where PLAN preserved restraint but B did not name it.
- Retain layer-3 consequence checks; they were the strongest discriminator.

## 5. Methodology caveats

- **Fresh-context fidelity:** Reconstruction was frozen and not modified. Validity audit found no contamination signs.
- **Single-run limitation:** One run only; no variance estimate.
- **Borderline capture calls:** #7, #90, and #93 were scored captured inclusively. Only #87 was missed, and it has no feature-level why.
- **System-level cross-cutting:** S4 is the subjective case; see the appendix above.
- **Confabulation:** No ungrounded reconstruction cases were found.
- **Evidence-bound denials:** Several rule recoveries were denied because rationale was absent, especially F5, F19, F23, F29, F33, and F39.
- **Operational compromise:** TIMING.json had phase 1 and phase 2A only; phase 2B timing was omitted. Run label is blank because wave metadata was outside the prompt allowlist.

End of report.
