# REPORT - CARE run 001
> Phase 2B evidence-bound scoring for Pocket Aviary v1. Frozen reconstruction was scored as written; missing rationale was not backfilled.
## 1. Headline
| Score | Value |
|---|---:|
| Planning quality | **64.2%** |
| Intent fidelity | **46.0%** |
| Combined quality | **6363** |

**Diagnostic split:**

- System-level fidelity: **57.1%**
- Feature-level fidelity: **42.9%**
- (Planning, fidelity) coordinate: `(64.2, 46.0)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label | (blank) |
| Timestamp | 2026-05-09T14:25:23Z |
| Candidate model | gemini-3-flash-preview |
| Candidate effort | medium |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **77 / 120** = **64.2%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 5 | 83.3% |
| concepts.md | 4 | 2 | 50.0% |
| bird_engine.md | 22 | 17 | 77.3% |
| interactions.md | 20 | 13 | 65.0% |
| aviary_layout.md | 18 | 7 | 38.9% |
| accounts_sync.md | 18 | 11 | 61.1% |
| social_optional.md | 10 | 5 | 50.0% |
| accessibility_perf.md | 18 | 13 | 72.2% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **77** | **64.2%** |

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | `product_brief.md` | yes | Captured: browser-based virtual aviary where birds evolve from presence and small interactions. |
| 2 | "Feels alive, not robotic" design-philosophy section | `product_brief.md` | yes | Captured as an explicit scope boundary and risk theme. |
| 3 | "Notice, never announce" principle callout | `product_brief.md` | yes | Captured as an explicit scope boundary. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | `product_brief.md` | no | Naturalist surfaces appear, but the matter-of-fact system/error exception is missing. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | `product_brief.md` | yes | Captured in non-goals. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | `product_brief.md` | yes | Captured through Day 0 two birds and birds 3-7 age gating. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | `concepts.md` | no | No glossary or domain-term layer. |
| 8 | Definition of "presence" (idle attention as interaction) | `concepts.md` | yes | Captured with visible + focused + activity precision. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | `concepts.md` | yes | Borderline: vector and mood systems are separate, but timescale language is compressed. |
| 10 | Definition of "settle" as user-initiated session end | `concepts.md` | no | Settle is named but not defined as session-end semantics. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | `bird_engine.md` | yes | Captured with all five traits. |
| 12 | Personality drift function (low-pass filter) | `bird_engine.md` | yes | Captured as filtered presence signals. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | `bird_engine.md` | yes | Captured explicitly. |
| 14 | Personality drift is monotonic toward expressive, never punishing | `bird_engine.md` | yes | Captured as values never decrease due to neglect. |
| 15 | Mood state (fast-timescale, resets daily-ish) | `bird_engine.md` | yes | Borderline: mood enum and transitions captured; daily-ish reset not explicit. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | `bird_engine.md` | yes | Captured for time of day, recent interactions, and personality; ambient weather input is thin. |
| 17 | Procedural call grammar (motifs combined at runtime) | `bird_engine.md` | yes | Captured with motif library and WebAudio synthesis. |
| 18 | Per-bird call signature (recognizable by ear) | `bird_engine.md` | no | Species motifs appear, but per-bird recognizability does not. |
| 19 | Chorus mixing (real chorus, not stacked loops) | `bird_engine.md` | no | Listen-in mix appears, but chorus mixing is not specified. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | `bird_engine.md` | yes | Captured through vocal_frequency shaping cadence. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | `bird_engine.md` | yes | Captured with preening/head-tilting and continuous ambient motion. |
| 22 | Mood-shaped idle motion | `bird_engine.md` | no | Mood shapes audio and mood state, not visible idle motion. |
| 23 | Bird species pool for v1 (~6 species) | `bird_engine.md` | yes | Captured as species pool of 6. |
| 24 | Bird naming (user-assigned at adoption; renameable) | `bird_engine.md` | yes | Captured for user-assigned names; renameability is not explicit. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | `bird_engine.md` | yes | Borderline: Day 0 two birds captured; auto-selected signup flow and catalog refusal absent. |
| 26 | Maximum 7 birds per aviary | `bird_engine.md` | yes | Captured through birds 3-7 gating. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | `bird_engine.md` | yes | Captured explicitly as age-based and not engagement. |
| 28 | Personality vector persistence (server-side, never resets) | `bird_engine.md` | yes | Captured through server ownership and canonical state; never-reset rationale absent. |
| 29 | Mood persistence across sessions | `bird_engine.md` | yes | Captured by stored current_mood and server-authored canonical snapshots. |
| 30 | Bird-to-bird interaction (calls and reactions) | `bird_engine.md` | no | Not specified. |
| 31 | Bird identity stability (stable internal id) | `bird_engine.md` | yes | Captured as stable bird_id. |
| 32 | Personality vector exposure (NEVER shown numerically) | `bird_engine.md` | no | No UI exposure prohibition. |
| 33 | Return-greeting on viewer arrival | `interactions.md` | yes | Captured with fresh session/visibility return greeting. |
| 34 | Greeting variation by absence length | `interactions.md` | no | Only a >15 minute trigger appears; no absence-length variation. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | `interactions.md` | yes | Captured through boldness-based selection. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | `interactions.md` | yes | Captured with staggered randomized offsets. |
| 37 | No "Welcome back!" toast or banner | `interactions.md` | yes | Captured explicitly. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | `interactions.md` | yes | Captured through gain-node rebalancing. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | `interactions.md` | yes | Captured by retaining ambient aviary noise. |
| 40 | Offer interaction (seed, song fragment, still pool) | `interactions.md` | yes | Captured in the in-scope list as Seed, Song, Pool. |
| 41 | Offer reaction varies by bird mood and curiosity | `interactions.md` | no | Only offer success nudging content is described; curiosity variation absent. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | `interactions.md` | no | Not specified. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | `interactions.md` | yes | Borderline: Settle is named, but evening shift and session-end semantics are absent. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | `interactions.md` | no | Not specified. |
| 45 | Field notebook auto-entries (specific naturalist tone) | `interactions.md` | yes | Captured through Naturalist prose ID and contextual bird/event data. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | `interactions.md` | no | Not specified. |
| 47 | Field notebook is read-only (user cannot edit entries) | `interactions.md` | no | Not specified. |
| 48 | Presence accounting (idle attention counted as interaction) | `interactions.md` | yes | Captured. |
| 49 | Presence accounting requires tab focus + cursor + visibility | `interactions.md` | yes | Captured as visible + focused + activity. |
| 50 | No streak counter, no "days visited" display | `interactions.md` | yes | Captured in no-gamification non-goals. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | `interactions.md` | yes | Borderline: client pulls while visible and server tick continues; explicit render pause absent. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | `interactions.md` | no | Not specified. |
| 53 | Single horizontal scene (one screen, no panning) | `aviary_layout.md` | yes | Captured as single horizontal responsive scene. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | `aviary_layout.md` | yes | Captured as three perch zones with z-depth. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | `aviary_layout.md` | no | Not specified. |
| 56 | Day/night cycle tied to user local time | `aviary_layout.md` | yes | Captured through local-time day/night and solar position. |
| 57 | Evening palette shift (warmer hues; calls quieter) | `aviary_layout.md` | no | Not specified. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | `aviary_layout.md` | no | Not specified. |
| 59 | Ambient weather (rare passing rain; soft wind) | `aviary_layout.md` | yes | Captured as ambient weather, with no rarity/detail. |
| 60 | Weather affects mood (rain dampens vocal frequency) | `aviary_layout.md` | no | Not specified. |
| 61 | Ambient leaf/feather drift motion | `aviary_layout.md` | yes | Captured explicitly. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | `aviary_layout.md` | no | Not specified. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | `aviary_layout.md` | no | Not specified. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | `aviary_layout.md` | no | Not specified. |
| 65 | Top bar auto-fades when cursor is idle | `aviary_layout.md` | no | Not specified. |
| 66 | Aviary scene loads with motion already in progress | `aviary_layout.md` | yes | Captured by first snapshot frame and ambient motion. |
| 67 | Loading state is a quiet field, not a spinner | `aviary_layout.md` | yes | Captured explicitly. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | `aviary_layout.md` | no | Not specified. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | `aviary_layout.md` | no | Not specified. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | `aviary_layout.md` | no | Responsive canvas appears, but no never-crop rule. |
| 71 | Email + magic-link sign-in (no passwords) | `accounts_sync.md` | yes | Captured as magic-link email auth. |
| 72 | Magic link expiry (15 minutes) | `accounts_sync.md` | no | Not specified. |
| 73 | Single-user accounts (one aviary per account at v1) | `accounts_sync.md` | yes | Captured by account to aviary_id and v1 scope. |
| 74 | Synthetic account ID (not email-derived) for internal references | `accounts_sync.md` | yes | Captured as UUID primary key with encrypted email. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | `accounts_sync.md` | yes | Captured explicitly. |
| 76 | Client pulls state snapshot on visibility | `accounts_sync.md` | yes | Captured. |
| 77 | Client interpolates between snapshots for smooth motion | `accounts_sync.md` | yes | Captured. |
| 78 | Multi-device sync (state is canonical server-side) | `accounts_sync.md` | yes | Captured. |
| 79 | Last-write-wins is forbidden for personality state | `accounts_sync.md` | yes | Captured semantically by clients never writing state and append-only events. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | `accounts_sync.md` | yes | Captured. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | `accounts_sync.md` | no | Not specified. |
| 82 | Per-device session token (revocable from settings) | `accounts_sync.md` | no | Session token issuance appears, but no per-device/revocation model. |
| 83 | Account export (download a JSON snapshot of your aviary) | `accounts_sync.md` | no | Not specified. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | `accounts_sync.md` | no | Not specified. |
| 85 | No telemetry on per-bird interactions for ML model training | `accounts_sync.md` | yes | Borderline: aggregate telemetry excludes bird-state/interaction history, but ML training is not named. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | `accounts_sync.md` | yes | Captured. |
| 87 | Privacy policy link in account settings | `accounts_sync.md` | no | Not specified. |
| 88 | Email change flow (verify new address before switching) | `accounts_sync.md` | no | Not specified. |
| 89 | Visit invitations (email-based, opt-in per invite) | `social_optional.md` | yes | Captured as one-time opt-in invitations; email basis not explicit. |
| 90 | Visits default OFF for new accounts | `social_optional.md` | yes | Captured semantically by opt-in only. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | `social_optional.md` | yes | Captured as read-only visits. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | `social_optional.md` | yes | Captured by read-only constraint. |
| 93 | No chat, no comments, no avatars during visits | `social_optional.md` | no | No social network features appears, but chat/comments/avatars are not specified. |
| 94 | No "your friend visited!" notification by default | `social_optional.md` | no | Not specified. |
| 95 | Visit revocation (host can revoke invite at any time) | `social_optional.md` | no | Not specified. |
| 96 | Visit log (host can see who visited and when, in account settings) | `social_optional.md` | no | Not specified. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | `social_optional.md` | no | Not specified. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | `social_optional.md` | yes | Captured through no public discovery/social network features. |
| 99 | Screen-reader narration of aviary state (running prose) | `accessibility_perf.md` | yes | Captured. |
| 100 | Narration cadence is slow (no overwhelming the SR) | `accessibility_perf.md` | no | Not specified. |
| 101 | Narration prose is naturalist, not announcement-style | `accessibility_perf.md` | yes | Captured. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | `accessibility_perf.md` | yes | Captured. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | `accessibility_perf.md` | yes | Borderline: cross-fade mode suggests designed alternative, but charm rationale is thin. |
| 104 | Captioning toggle for procedural calls (text describes mood) | `accessibility_perf.md` | yes | Captured as procedural call captions; toggle is not explicit. |
| 105 | WCAG AA contrast on all user-copy surfaces | `accessibility_perf.md` | no | Not specified. |
| 106 | Keyboard-only navigation through all interactive surfaces | `accessibility_perf.md` | yes | Captured. |
| 107 | Focus indicators visible against the aviary background | `accessibility_perf.md` | no | Not specified. |
| 108 | Initial JS bundle <2MB | `accessibility_perf.md` | yes | Captured. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | `accessibility_perf.md` | yes | Captured. |
| 110 | 60fps idle motion target on 5-year-old laptop | `accessibility_perf.md` | yes | Captured. |
| 111 | No memory growth over 30-minute session | `accessibility_perf.md` | yes | Captured. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | `accessibility_perf.md` | yes | Captured through WebAudio synthesis and motif delivery. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | `accessibility_perf.md` | yes | Captured. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | `accessibility_perf.md` | yes | Captured for operational telemetry and aggregate privacy boundary. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | `accessibility_perf.md` | no | Not specified. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | `accessibility_perf.md` | no | Not specified. |
| 117 | Out of scope: native mobile app | `non_goals.md` | yes | Captured. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | `non_goals.md` | yes | Captured. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | `non_goals.md` | yes | Captured. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | `non_goals.md` | yes | Captured. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **57.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:5: "Feels alive, not robotic" plus Heartbeat, ambient motion, reliable tick, naturalist-not-robotic narration. | PLAN.md:4,27,101-102,145-147 preserve alive feeling across scope, tick, motion/loading, sync/audio/accessibility risks. | yes | yes | no | full | The aliveness principle survived as a broad design constraint, with plan evidence across engine, rendering, loading, sync, audio, and accessibility. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md:3: "Notice, never announce" with no textual Welcome back and quiet field instead of spinner. | PLAN.md:4,16,93,102,140 apply no-announcement/no-engagement rules to scope, gamification, return greeting, loading, and bird unlocks. | yes | yes | no | partial | The main rule survived; the processed-vs-seen rationale and cumulative anti-toast argument did not. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md:49,65,69: field-notebook prose, naturalist screen-reader prose, and call captions as specific surfaces. | PLAN.md:48,54-59,114-115 preserve named birds, notebook context, narration prose, and low-trill captions. | yes | yes | no | partial | Specific naturalist surfaces survived, but the anti-generic charm rationale was only implicit. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md:9,16-18,100,139-140 constrain one scene, non-goals, two starters, and age-gated birds 3-7. | no | yes | yes | partial | The plan has restraint rules, but the reconstruction did not name restraint as a product rationale. |
| S5 - naturalist-voice-with-system-exception | 2 | included | none | none | no | no | yes | none | Naturalist voice survived, but the matter-of-fact system/error exception is absent from both plan and reconstruction. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:7,9,125: presence drives slow change, absence is not punished, and active presence beats open tabs. | PLAN.md:67-70,88,141,16-17 ground precision, drift, no punishment, validation, and anti-gamification. | yes | yes | no | partial | Presence-as-attention and precision survived; settle/tab-close equivalence did not. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:11,61,77,87: one truth across devices, server-owned state, clients never write. | PLAN.md:22-34,65-75,85-88 preserve server tick, snapshots, append-only events, and single source of truth. | yes | yes | no | partial | Canonical server ownership and sync coherence survived; the divergent-client/LWW failure story was compressed away. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md:13,93,147: PII partitioning and no bird-state or interaction history in aggregate telemetry. | PLAN.md:40-43,130-132,12,18 preserve encrypted email, aggregate telemetry boundary, opt-in social, and no social network. | yes | yes | no | full | The privacy boundary survived as a cross-cutting system concern, though feature-level privacy details are thinner. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:15: accessibility preserves naturalist voice rather than becoming a separate mechanical layer. | PLAN.md:11,113-119,147 include naturalist narration, captions, reduced-motion cross-fades, keyboard support, and accessibility-regression risk. | yes | yes | no | partial | Accessible surfaces remain designed and voiced, but v1 launch parity is not present. |

Multi-layer system-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | Y | Y | Y |
| S2 | Y | N | N |
| S6 | Y | Y | N |
| S7 | Y | Y | N |
| S9 | Y | Y | N |

**Cross-cutting evidence appendix:**

- S1: PLAN.md:4 scope; 27 and 65-75 tick; 101-102 motion/loading; 145-147 sync/audio/accessibility risks.
- S2: PLAN.md:4 principle; 16 no gamification; 93 no textual welcome; 102 quiet field; 140 age not engagement.
- S3: PLAN.md:48 names; 54-59 notebook context; 114-115 naturalist narration/captions.
- S4: PLAN.md:9 one scene; 16-18 non-goals; 100 horizontal canvas; 139-140 starter pair and birds 3-7.
- S5: PLAN.md:114-115 naturalist narration/captions, but no matter-of-fact system exception.
- S6: PLAN.md:67-70 drift/monotonicity; 88 presence precision; 16-17 no gamification/Tamagotchi; 141 validation.
- S7: PLAN.md:22-34 server canonical split; 65-75 tick; 85-88 snapshots/no conflicts; 145 sync continuity risk.
- S8: PLAN.md:40-43 PII partition; 130-132 telemetry boundary; 12 and 18 opt-in/no social network.
- S9: PLAN.md:11 accessibility in scope; 113-119 narration/captions/reduced motion/keyboard; 147 accessibility regression risk.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **42.9%**.

Reachable feature-level whys: **29 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md:125: records presence only when visible + focused + activity, active presence rather than open tabs. | PLAN.md:88: presence recorded only when visible + focused + activity; pings feed aggregation. | no | partial | L1 recovered; individual-signal failure cases and silent population drift corruption are absent. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:105-107,157: filtered presence deltas, one-week/three-week calibration, toy-vs-screensaver risk. | PLAN.md:67-70,144: filter over presence signals, one-week/three-week calibration, toy-vs-screensaver risk. | no | full | All three calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:9,109,155: absence is not punished; values never decrease due to neglect; drift monotonicity is validated. | PLAN.md:17,70,141: no distress, no punishment, monotonic drift validation. | no | partial | No-punishment survived; the two-weeks-away/quieter-not-mistrust consequence did not. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:31,115-117,161: motif/WebAudio calls and avoidance of phase-canceling or repetitive artifacts. | PLAN.md:77-79,105-107,146: motif library, WebAudio synthesis, fallback, and audio-uncanniness risk. | no | partial | Procedural audio and artifact avoidance survived; audio-as-affective-spine/cascade did not fully surface. |
| F5 | mood-shaped-idle-motion | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: mood does not shape visible idle motion. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | The cap/range appears through birds 3-7, but not the recognizability rationale. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md:77,87,123: server owns personality/mood/drift; clients never write state. | PLAN.md:33,67-75,87: server owns vectors, applies additive deltas, and is single source of truth. | no | partial | Canonical server persistence and sync implications survived; deleting-the-known-bird rationale did not. |
| F8 | personality-vector-never-numerical | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: no prohibition on numeric trait display. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md:35: one bird picked by boldness and mood, staggered, procedural, no textual Welcome back. | PLAN.md:90-93: fresh-session trigger, one-bird algorithm, staggered responses, procedural/no textual welcome. | no | partial | One-bird procedural greeting survived; absence-length calibration and wrong-product consequence mostly did not. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md:3,35: no textual Welcome back; return greeting is procedural visual/audio. | PLAN.md:4,90-93: Notice, never announce plus no textual Welcome back. | no | partial | The primary no-toast rationale survived; temptation and adjacent days-gone variants were not recovered. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: closing-tab equivalence absent. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECONSTRUCTION.md:49: timestamps, Naturalist prose ID, bird names, events as naturalist prose records. | PLAN.md:54-59,114: notebook stores naturalist prose templates and contextual bird/event data. | no | partial | Naturalist observer prose survived; rare cadence and read-only/not-feed rationale did not. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md:125: visible + focused + activity, active presence rather than open tabs. | PLAN.md:88: visible + focused + activity in last 3 minutes. | no | partial | Conjunction rule survived; individual-signal edge cases and silent drift corruption did not. |
| F14 | no-streak-counter | 4 | yes | included | none | none | yes | none | No-streak/no-gamification rule appears, but none of the presence-for-birds rationale is reconstructed. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md:133: first snapshot, cold-cache quiet field rather than spinner, preserving quiet product voice. | PLAN.md:101-102: continuous motion; render first snapshot; quiet field rather than spinner. | no | full | The continuing-aviary/loading-state rationale was recovered cleanly. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md:93: account_id primary key, email_encrypted, privacy boundary around PII. | PLAN.md:40-43: UUID primary key used elsewhere; encrypted email. | no | partial | UUID/PII rationale survived; retrofit impossibility did not. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md:63,77,87: Heartbeat tick updates active aviaries; server owns canonical tick and state. | PLAN.md:27,33,65-75,87: periodic tick, server ownership, event log, canonical snapshots. | no | partial | Tick and coherence survived; client-side collapse failure mode did not. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md:105,123: additive deltas and clients never write state, only append events. | PLAN.md:67-70,87: additive deltas; clients never write state and append event log. | no | partial | Implementation rule survived; silent deletion/lost-drift failure story did not. |
| F19 | sync-conflict-matter-of-fact | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: sync conflict tone absent. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md:147: excludes bird-state and interaction history from aggregate telemetry. | PLAN.md:130-132: operational telemetry plus no bird-state or interaction history in aggregate telemetry. | no | partial | Technical telemetry boundary survived; ML-training and relationship-as-private-data rationale did not. |
| F21 | visit-read-only-ambient | 2 | yes | included | none | none | yes | none | Read-only visits appear, but the no-co-presence/multi-user-simulation rationale is missing. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | none | none | yes | none | No public discovery survived, but the comparison/ownership rationale did not. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:15,65: naturalist screen-reader prose, same voice rather than mechanical layer. | PLAN.md:11,113-115,147: naturalist narration, prose example, and regression risk. | no | partial | Running naturalist prose and same-product voice survived; ARIA automation/user-event priority did not. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md:67: 2-second cross-fades between static poses, preserving experience with less motion. | PLAN.md:11,116: reduced-motion mode uses pose cross-fades. | no | partial | Designed cross-fade alternative survived; calls/drift/notebook parity and stripped-fallback consequence did not. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md:141: first bird under 500ms supports alive feeling. | PLAN.md:127: Time-to-First-Bird under 500ms on 4G/mid-tier. | no | full | The affective performance threshold survived. |
| F27 | no-gamification | 4 | yes | included | none | none | yes | none | The no-gamification rule is present, but the counter/reward-loop rationale is absent. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md:9: absence is not punished; birds do not die, hunger, or show distress. | PLAN.md:17,70: birds do not die/hunger/show distress; values never decrease due to neglect. | no | full | The observational/no-punishment rationale survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Starter pair is captured only thinly; arrivals-not-catalog rationale is absent and reconstruction says not recoverable. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md:153: birds 3-7 by account age not engagement, preserving no gamification and no punishment for absence. | PLAN.md:140: birds 3-7 unlocked by account age, not engagement. | no | partial | The rejection of engagement rewards survived; relationship-deepening and economy-collapse layers did not. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | Plan has stable bird_id, but reconstruction explicitly says bird stable identity is not recoverable from plan. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | Stored current_mood suggests the rule, but the continued-while-gone rationale is absent. |
| F33 | field-notebook-read-only-observer-record | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured. |
| F34 | account-export-relationship-copy | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | none | none | yes | none | Aggregate telemetry rule appears, but the observability-as-back-door rationale is absent. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Opt-in read-only invitations survive, but private-relationship/not-publishing rationale is absent. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured. |
| F39 | visitor-sees-actual-aviary | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured. |
| F40 | sr-narration-cadence-slow | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: narration cadence absent. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | Y | N | N |
| F2 | Y | Y | Y |
| F3 | Y | Y | N |
| F4 | Y | Y | N |
| F7 | Y | N | Y |
| F9 | Y | Y | N |
| F10 | Y | N | N |
| F12 | Y | N | N |
| F13 | Y | N | N |
| F14 | N | N | N |
| F15 | Y | Y | Y |
| F16 | Y | Y | N |
| F17 | Y | Y | N |
| F18 | Y | N | Y |
| F20 | N | N | Y |
| F24 | Y | Y | N |
| F25 | Y | N | N |
| F27 | N | N | N |
| F30 | N | Y | N |
| F31 | N | N | N |
| F35 | - | - | - |
| F40 | - | - | - |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| `Possible gold whys` | 49 | From gold_why_totals |
| `Possible total weight` | 152 | Total possible weighted denominator |
| `Reachable gold whys` | 38 | Included S whys plus captured F whys |
| `Excluded unreachable feature whys` | 11 | Feature anchors not captured |
| `Recovered / reachable weight` | 58.0 / 126.0 | Included weighted recovery |
| `Whys with reconstruction evidence` | 26 | Rows with non-none reconstruction rationale evidence |
| `Whys with PLAN grounding` | 27 | Rows with non-none plan rationale grounding |
| `rule_without_why cases` | 12 | Mechanism survived without rationale |
| `plan_only_not_reconstructed cases` | 1 | Plan carried the principle but reconstruction did not |
| `ungrounded_reconstruction cases` | 0 | Reconstruction rationale not grounded in PLAN |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 48.0 | 26.0 | 54.2% |
| Affective whys | 78.0 | 32.0 | 41.0% |
| Weight 2 whys | 26.0 | 8.0 | 30.8% |
| Weight 3 whys | 100.0 | 50.0 | 50.0% |
| System-level whys | 28.0 | 16.0 | 57.1% |
| Feature-level whys (reachable) | 98.0 | 42.0 | 42.9% |

---

## 3. Diagnostic patterns

- **Affective vs functional:** Functional whys recovered better (26.0/48.0) than affective whys (32.0/78.0). The plan preserved server-side state, drift calibration, and presence precision better than the relationship/protection rationales behind social, privacy, and restraint features.
- **Weight-3 vs weight-2:** Weight-3 whys recovered 50.0/100.0, while weight-2 whys recovered 8.0/26.0. The strongest architecture and calibration signals were preserved; smaller exception rationales were often reduced to rules.
- **System-level vs feature-level:** System-level fidelity (57.1%) exceeded feature-level fidelity (42.9%). The broad philosophy survived, but local whys often lost their downstream consequence layer.
- **Multi-layer recovery:** L3/downstream consequences were most often dropped: F3, F7, F17, F18, F24, F25, F30, S6, S7, and S9 all lost or weakened their consequence layer.
- **Subdomain patterns:** Engine/sync/performance were strongest; social optional, account lifecycle, restraint, and field notebook governance were weakest.
- **Evidence-bound effects:** Ten reachable feature whys were rule-without-why cases. Most would look acceptable in an implementation review, but v06 correctly denied intent credit.

The failure shape suggests a compact planning style that is good at preserving buildable mechanisms and poor at carrying the affective/legal/product reasons those mechanisms exist.

## 4. Recommendations for v2 hardening

- Keep targeted headroom whys like F29-F40; they exposed real compression that the original feature set might have missed.
- Add more relationship-ownership and social/privacy exception whys. This run captured rules like opt-in visits and aggregate telemetry but often lost why those rules exist.
- Preserve multi-layer scoring for high-weight whys. The layer split made downstream consequence loss visible rather than hiding it inside partial semantic matches.
- Add a report-level rollup for rule-only recoveries in future benchmark dashboards, since this is the dominant failure pattern here.
- Clarify examples for plan-only system partials, because S4-style restraint can be cross-cutting in PLAN even when the reconstruction does not name it.

## 5. Methodology caveats

- **Fresh-context fidelity:** The phase-2B prompt states fresh context and the reconstruction was treated as frozen. I did not read `prd/`, peer slots, or other waves.
- **Single-run limitation:** This is one candidate run with no variance signal.
- **Borderline capture calls:** Inclusive capture affected mood state, starter birds, settle naming, background-tab behavior, per-bird ML telemetry, and reduced-motion charm. Stricter capture would lower planning quality and shrink the feature-why denominator.
- **System-level cross-cutting:** S3 and S4 were the most subjective cross-cutting calls. The appendix lists the concrete plan sites used for each judgment.
- **Confabulation cases:** No clear ungrounded-reconstruction cases were counted. Some reconstruction prose generalized from the plan, but did not introduce obvious external facts.
- **Evidence-bound denials:** F6, F14, F21, F23, F27, F29, F31, F32, F36, and F37 were denied recovery because the why was missing even though the rule was present.

End of report.
