# REPORT — CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Exact frozen-reconstruction evidence and PLAN grounding are recorded for every S1-S9 and F1-F40 row.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **100.0%** |
| Intent fidelity | **84.2%** |
| Combined quality | **9984** |

**Diagnostic split:**

- System-level fidelity: **92.9%**
- Feature-level fidelity: **82.3%**

**(Planning, fidelity) coordinate:** `(100.0, 84.2)` — plot on a 2D scatter with both axes 0-100; upper-right is best.


### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-08-12T23:57:08Z |
| Candidate model | grok-4.6 |
| Candidate effort | high |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **120 / 120** = **100.0%**.

By PRD file:

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
| 1 | Headline product concept statement | product_brief.md | yes | Captured by PLAN. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured by PLAN. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by PLAN. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by PLAN. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured by PLAN. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by PLAN. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured by PLAN. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured by PLAN. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by PLAN. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by PLAN. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured by PLAN. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by PLAN. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by PLAN. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by PLAN. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by PLAN. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by PLAN. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by PLAN. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by PLAN. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by PLAN. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by PLAN. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by PLAN. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by PLAN. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by PLAN. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by PLAN. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured by PLAN. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by PLAN. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by PLAN. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by PLAN. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by PLAN. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by PLAN. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by PLAN. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by PLAN. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by PLAN. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by PLAN. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by PLAN. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured by PLAN. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by PLAN. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by PLAN. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured by PLAN. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by PLAN. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by PLAN. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by PLAN. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by PLAN. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured by PLAN. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by PLAN. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by PLAN. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by PLAN. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured by PLAN. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by PLAN. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by PLAN. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured by PLAN. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured by PLAN. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by PLAN. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by PLAN. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by PLAN. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured by PLAN. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured by PLAN. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by PLAN. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by PLAN. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by PLAN. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by PLAN. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured by PLAN. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured by PLAN. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by PLAN. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured by PLAN. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by PLAN. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by PLAN. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured by PLAN. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured by PLAN. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by PLAN. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by PLAN. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by PLAN. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by PLAN. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured by PLAN. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured by PLAN. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by PLAN. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by PLAN. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured by PLAN. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured by PLAN. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by PLAN. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by PLAN. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by PLAN. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured by PLAN. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured by PLAN. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by PLAN. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured by PLAN. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured by PLAN. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by PLAN. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured by PLAN. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by PLAN. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured by PLAN. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by PLAN. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by PLAN. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured by PLAN. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured by PLAN. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured by PLAN. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Captured by PLAN. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured by PLAN. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured by PLAN. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured by PLAN. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by PLAN. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured by PLAN. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured by PLAN. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured by PLAN. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by PLAN. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by PLAN. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured by PLAN. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by PLAN. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured by PLAN. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by PLAN. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by PLAN. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by PLAN. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by PLAN. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by PLAN. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by PLAN. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured by PLAN. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by PLAN. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured by PLAN. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured by PLAN. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by PLAN. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **92.9%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 — feels-alive-not-robotic | 4 | included | System intent: "Preserve the conceit that the aviary is already alive"; boot forbids "spinner," "logo splash," and "fade-from-black." | PLAN §§7.1,15.5: quiet field only; first frame mid-action; load-state leakage contradicts "already alive." | yes | yes | no | full | Aliveness is recovered across boot, procedural audio, motion, loading, and accessibility surfaces. |
| S2 — notice-never-announce | 4 | included | System intent: "Reject game and announcement pressure"; bans "welcome toasts," "visit badges," and says "Build the quiet window, not a game around it." | PLAN §§1.2,5.8,7.5,15.6: no welcome toast, no badges, no visit pings, no "you have been gone" surfaces. | yes | yes | no | partial | The rule and cumulative anti-announcement pressure survived, but the processed-vs-seen affective layer was not reconstructed. |
| S3 — charm-from-specificity | 2 | included | Feature whys: narration is "lowercase, present-tense, specific"; notebook inputs are "facts, never user-behavior tallies." | PLAN §§5.11,9.1,16: naturalist prose, named birds, no trait numbers, normative bird/call/listen-in vocabulary. | yes | yes | no | full | The reconstruction preserved specificity through prose, narration, captions, vocabulary, and anti-number surfaces. |
| S4 — restraint-over-richness | 2 | included | System intent: "Prefer restraint" and "When torn between a richer feature and restraint, cut the feature." | PLAN §§1.1,7.2,8.1,14.2: two starters, cap seven, one horizontal scene, no pan/zoom, ABX cap discipline. | yes | yes | no | full | The plan and reconstruction treat restraint as a cross-cutting product constraint. |
| S5 — naturalist-voice-with-system-exception | 2 | included | System intent: "Use two deliberate voices"; product surfaces use naturalist voice and system surfaces use matter-of-fact voice. | PLAN §§2.2,4.4,9.1,12,16: prose package, system error copy, matter-of-fact mail, no naturalist auth failures. | yes | yes | no | full | The voice split is explicit and inherited by auth, sync, a11y, notebook, narration, and mail. |
| S6 — presence-is-real-interaction | 4 | included | System intent: "Treat watching as the product"; presence uses the three-signal conjunction, union-not-sum, and host-only credit. | PLAN D1-D3, §§5.2,5.3,11: watching without moving, visible+focus+activity, overnight fixture, settle/session_end closes presence. | yes | yes | no | full | Precision, inflation prevention, and settle/tab-close equivalence all survived. |
| S7 — simulation-runs-server-side | 4 | included | System intent: "Keep the server as the source of truth"; "Every host device is a projector"; browser never advances canonical state. | PLAN §§2.1,2.2,5.1,6.1,6.3: sim tick is the only writer; clients render snapshots; no CRDT/LWW personality. | yes | yes | no | full | Server tick, multi-device coherence, and no-LWW failure avoidance are all recovered. |
| S8 — privacy-first-on-bird-data | 2 | included | System intent: "Make privacy an architectural boundary" with "No analytics role" and UUID-only logs/keys/files. | PLAN §§2.4,10.2,10.3,12: sim/ops split, aggregate-only telemetry, no per-bird funnel, no third-party replay. | yes | yes | no | full | The privacy stance is architectural, not merely policy-level. |
| S9 — accessibility-as-first-class-surface | 4 | included | System intent: "Ship accessibility as the same product"; reduced motion is "a designed aesthetic" and not "v1.1." | PLAN §§1.1,6.6,9,13.4,14.1: narration, reduced-motion renderer, captions, keyboard, contrast, launch gates. | yes | yes | no | full | The accessible surfaces preserve the affective core and ship with v1. |

For multi-layer system-level whys:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | ✓ | ✓ | ✓ |
| S2 | ✓ | ✗ | ✓ |
| S6 | ✓ | ✓ | ✓ |
| S7 | ✓ | ✓ | ✓ |
| S9 | ✓ | ✓ | ✓ |

**Cross-cutting evidence appendix (RUBRIC §5.1).**
- S1: PLAN §§7.1,15.5: quiet field only; first frame mid-action; load-state leakage contradicts "already alive."
- S2: PLAN §§1.2,5.8,7.5,15.6: no welcome toast, no badges, no visit pings, no "you have been gone" surfaces.
- S3: PLAN §§5.11,9.1,16: naturalist prose, named birds, no trait numbers, normative bird/call/listen-in vocabulary.
- S4: PLAN §§1.1,7.2,8.1,14.2: two starters, cap seven, one horizontal scene, no pan/zoom, ABX cap discipline.
- S5: PLAN §§2.2,4.4,9.1,12,16: prose package, system error copy, matter-of-fact mail, no naturalist auth failures.
- S6: PLAN D1-D3, §§5.2,5.3,11: watching without moving, visible+focus+activity, overnight fixture, settle/session_end closes presence.
- S7: PLAN §§2.1,2.2,5.1,6.1,6.3: sim tick is the only writer; clients render snapshots; no CRDT/LWW personality.
- S8: PLAN §§2.4,10.2,10.3,12: sim/ops split, aggregate-only telemetry, no per-bird funnel, no third-party replay.
- S9: PLAN §§1.1,6.6,9,13.4,14.1: narration, reduced-motion renderer, captions, keyboard, contrast, launch gates.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **82.3%**.

Reachable feature-level whys: **40 / 40** (the rest had unreachable anchors because the feature was not captured).

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | Presence monitor is "load-bearing"; presence true iff visible, focused, and recent activity; overnight-tab fixture gives 0 drift. | PLAN D1-D3, §§5.2,11: three signals, 4-minute activity window, union-not-sum, open-tab overnight fixture. | no | full | The precise conjunction, shortcut failures, and drift-corruption stakes were recovered. |
| F2 | drift-function | 4 | yes | included | "Drift function" says traits move only upward, calibration fixtures are not product analytics, and risks name "Tamagotchi numbers" vs "screensaver." | PLAN §§5.3,15.1: low-pass additive deltas, week-1/week-3 fixtures, daily caps, too-fast/too-slow risk. | no | full | All calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | System intent: "ambient, not distressed"; expressiveness gate makes birds "quieter, not warier" and absence "quiet, not failure." | PLAN §§3.3,5.3,15.1,16: traits never decrease; neglect changes gate; no Tamagotchi punishment. | no | full | The no-punishment exception is explicit in engine and product language. |
| F4 | procedural-call-grammar | 4 | yes | included | Procedural calls replace PCM, avoid recorded fallback, support recognizable identity, and chorus mix avoids phase-locking. | PLAN §§5.7,8.1,8.2,8.3,15.3: motif plans, WebAudio, no MP3/OGG sprites, detune/delay, audio uncanniness risk. | no | full | Procedural variation, chorus dependency, and WebAudio/fallback cascade survived. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | Rule only: idle micro-motion uses mood-linked clips such as preen, scan, tilt, shuffle, rest, and call. | Rule only: PLAN §7.3 maps motion kinds to mood examples and first-frame mid-action. | yes | none | The user-readable mood-without-label rationale was not reconstructed. |
| F6 | bird-count-cap-7 | 2 | yes | included | Recognizability test: same-bird ID must be >=80%; if it fails, "we do not raise the cap; we retune motifs." | PLAN §§1.1,8.1,14.2: cap seven; ABX recognizability; lower cap if chorus blurs. | no | full | The empirical recognizability ceiling survived. |
| F7 | personality-vector-persistence | 4 | yes | included | Canonicality: every device is a projector; clients never send trait values; personality has no CRDT/LWW document. | PLAN §§3.2,5.1,6.1,6.3,15.2: stored traits, sim tick writer, additive deltas, review rule "who writes this column?" | no | partial | Persistence and downstream sync rules survived; the affective consequence of losing a vector as deleting the known bird did not. |
| F8 | personality-vector-never-numerical | 2 | yes | included | Rule only: traits are allowed only in JSON takeout and raw traits are absent from snapshots/UI. | Rule only: PLAN D20, §§3.5,6.5,9.5: export is sole trait serialization; no ARIA stats; no personality-number UI. | yes | none | The stat-management/relationship-collapse why was not reconstructed. |
| F9 | return-greeting | 4 | yes | included | Return-greeting uses absence bands and bird scores so short return, long return, and second-device openings differ without toast or "you have been gone." | PLAN D16, §§5.8,7.1,13.2: one bird, absence bands, boldness/social warmth, procedural variation, no toast. | no | full | The bird-notices-you anchor and anti-canned variation survived. |
| F10 | no-welcome-back-toast | 4 | yes | included | Rejects "welcome toasts" and no "welcome back" anywhere; no "you have been gone" surface. | PLAN §§1.2,4.4,5.8,13.2: no welcome toasts/modals/banners; bird greeting only; no welcome string in product routes. | no | partial | The explicit ban and variants survived; the well-meaning-toast temptation / exact reframing layer did not. |
| F11 | settle-is-opt-in | 2 | yes | included | Presence reconstruction closes windows on settle/session_end; team notes say "The user never owes the birds a visit. Absence is quiet, not failure." | PLAN §§1.1,5.2,5.3,16: settle or close ends presence; settle gives zero trait delta; absence is not penalized. | no | full | The no-required-goodbye/no-penalty rationale is recovered semantically. |
| F12 | field-notebook-auto-entries | 4 | yes | included | Notebook compiler uses facts, never user-behavior tallies; entries remain observations rather than a session log; examples are lowercase present-tense. | PLAN §§3.6,5.11,13.1: sparse token bucket, lowercase naturalist prose, forbidden facts, read-only served prose. | no | full | Naturalist prose, concentrated voice, rarity, and read-only observation all survived. |
| F13 | presence-accounting | 4 | yes | included | Presence ingest requires three true signals; partial pings are not upgraded; overnight-tab fixture gives 0 drift. | PLAN §§4.2,5.2,5.3,11: visible+focus+activity, 30s pings, gap closure, open-tab overnight test. | no | full | The implementation-level presence why survived. |
| F14 | no-streak-counter | 4 | yes | included | Plan rejects any ticket teaching "presence is for a counter"; forbids streaks, green-dot calendars, visit badges, and "visited every." | PLAN §§1.2,5.11,7.5,13.1,15.6: no streaks/counters, no badges, forbidden notebook phrases, no badge component. | no | full | The counter-displacement and adjacent-disguise layers survived. |
| F15 | scene-loads-with-motion | 4 | yes | included | Boot quiet field, no spinner/splash/fade, first frame mid-action; load-state leakage contradicts "already alive." | PLAN §§7.1,7.3,15.5: quiet field; hydrate mid-action; first pose samples motion_phase; spinner is forbidden. | no | full | Mid-action boot, snapshot-driven continuation, and quiet-field loading all survived. |
| F16 | synthetic-account-id | 4 | yes | included | UUIDv7 primary keys are "time-ordered, not email-derived"; email is encrypted once and logs/keys/files use UUIDs. | PLAN D11, §§2.4,3.1,12,15.7: email ciphertext/hash split, UUID log discipline, avoid PII partition keys. | no | partial | PII containment survived; the impossible-to-retrofit design-time rationale did not. |
| F17 | server-side-simulation-tick | 4 | yes | included | Tick worker transaction; server source of truth; browser never advances canonical state; no client sim prevents dual-device fork. | PLAN D4,D22, §§2.1,5.1,6.1,6.3: slow tick, server writer, client snapshots, no divergent simulations. | no | full | The architectural reason for the server tick survived. |
| F18 | no-last-write-wins-personality | 4 | yes | included | No CRDT, no LWW document, no client personality cache; rejecting trait keys and review question "who writes this column?" address stale-device failures. | PLAN §§3.4,6.1,6.3,13.3,15.2: additive deltas, append-only events, sim-only trait writes, two-device tests. | no | full | The no-LWW failure mode and implementation rule survived. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | Rule only: errors are matter-of-fact, no bird names, and no naturalist auth failures. | Rule only: PLAN §§4.4,9.5,12,16: matter-of-fact system copy and email templates. | yes | none | The evasive-charm/error-clarity rationale was not reconstructed. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | Privacy is architectural; deliberately unmeasured behavior excludes per-bird offers, listen-in, presence minutes, and population boldness. | PLAN §§2.4,10.2,10.3,15.7: telemetry pipelines never touch sim data; fixture accounts calibrate drift; no real-user warehouse rollups. | no | full | The relationship-data boundary and data-pipeline enforcement survived. |
| F21 | visit-read-only-ambient | 2 | yes | included | Visitor snapshot stripping keeps visits read-only; birds do not notice the visitor and the host snapshot is not mutated. | PLAN §§3.7,4.3: visitor snapshots omit greeting, offers, notebook, listen-in; no visitor events are inserted. | no | full | The observation-not-co-presence and no-accidental-drift rationale survived. |
| F22 | no-friend-visited-notification | 2 | yes | included | Visit notify is email-only/off by default; host is not told in-session; no host toast matches the no-announcement principle. | PLAN D14, §§4.3,7.5,15.6: silent visit log, no badge, no host ping, notification toggle off by default. | no | full | The attention-loop refusal is recovered through the no-announcement framing. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | Indexes and views must not rank aviaries; no public surface and no "just in case" aggregates. | PLAN §§1.2,3.8,10.3: no leaderboards/discovery/public aviaries; no ranking aggregates; no public metrics to expose later. | no | full | The comparison-surface refusal and architectural cascade survived, though compressed. |
| F24 | sr-narration-running-prose | 4 | yes | included | Screen-reader narration is one polite live region, avoids every micro-shift, and is "weather-to-climate of the scene." | PLAN §§9.1,9.5,13.4: naturalist prose, no ARIA stat automation, no static screenshot mode, live-region checks. | no | full | Running prose, same-product access, and implementation refusal all survived. |
| F25 | reduced-motion-mode | 4 | yes | included | Reduced motion is "a designed aesthetic"; it keeps greeting, audio, captions, and slower sun shifts; not "v1.1." | PLAN §§6.6,7.6,9,13.4,14.1: cross-fades, no ornaments, audio/captions unchanged, milestone before launch. | no | full | The accessible surface is recovered as a different rendering of the same aviary. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | First-bird budget is first canvas pixel and drives critical JS, snapshot size, font, and code-splitting choices. | PLAN §§7.1,10.1,10.4,14.3,15.5: 500ms gate, bird_visible mark, bootstrap snapshot, bundle budget. | no | full | The affective-performance bridge survived. |
| F27 | no-gamification | 4 | yes | included | Rejects scores, quests, streaks, badges, XP, green-dot calendars, milestone celebrations, engagement dashboards, and "streak-shaped funnels." | PLAN §§1.2,10.3,13.1,15.6,16: no counters, no badges, linter, no engagement dashboards, quiet window not game. | no | full | The refusal, temptation pressure, and creep-prevention layer survived. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | System intent says ambient, not distressed; after absence birds are "quieter, not wounded" and "Absence is quiet, not failure." | PLAN §§1.2,3.3,5.3,15.1,16: no death/hunger/distress, monotonic traits, expressiveness gate, no negative drift. | no | full | The non-custodial, non-punitive rationale survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | Rule only: first two birds use different silhouettes/motifs and complementary traits so they are "different animals, not twins." | Rule only: PLAN §§1.1,5.10: system picks two species; user names/renames; no species catalog. | yes | none | The arrived-animals-not-avatar-configuration why was not reconstructed. |
| F30 | age-based-new-bird-offers | 4 | yes | included | Age-gated adoption uses aviary age, not attention, for "Months-scale deepening"; the surface is naturalist, not a reward chest. | PLAN D15, §§5.10,14.1: age gates 56/120/210/300/420 days, no score/attention/paid tier, no confetti. | no | full | The anti-reward-loop unlock rationale survived. |
| F31 | stable-bird-identity | 4 | yes | included | Bird ids are "STABLE forever"; migrations may never delete and replace rows; identity is bird id, not species. | PLAN §§3.2,5.10,15.10: stable internal id, species edits remap in place, rename is UPDATE name. | no | full | Identity continuity and relationship preservation survived. |
| F32 | mood-persists-across-sessions | 2 | yes | included | Mood persists across sessions and never snaps to content on start, preserving the "already alive" conceit. | PLAN §§4.2,5.4,7.1,15.5: start pulls snapshot; mood persists; no neutral reset; quiet field continues. | no | full | The continuity rationale survived. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | Notebook entries remain observations rather than a session log; served fields are prose only and the notebook is read-only/sparse. | PLAN §§3.6,5.11,13.1: read-only served entries, allowed observation facts, forbidden user-behavior tallies. | no | full | The observer-record rationale survived, though compressed. |
| F34 | account-export-relationship-copy | 2 | yes | included | Rule only: account export JSON is emailed on demand and is the sole place traits are serialized to the user. | Rule only: PLAN §§1.1,4.1,6.5: export endpoint/job and JSON snapshot fields. | yes | none | The relationship-copy/right-to-take-it rationale was not reconstructed. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | Soft-delete keeps tick running "so restore is coherent"; hard-delete wipes all sim/account data and "No cold analytics copy exists to forget." | PLAN §§4.1,6.5,15.7: 30-day restore, hard-delete events/birds/snapshots/notebook/visits/sessions/accounts, no analytics copy. | no | partial | Privacy/hard-delete and total-wipe layers survived; accidental-regret relationship layer did not. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | Aggregate-only measurement excludes per-bird offers, listen-in, presence minutes, species popularity, visit frequency, and population boldness. | PLAN §§2.4,10.2,10.3,15.7: ops-only telemetry, no sim table analytics, no product funnels, no warehouse rollups. | no | full | The technical boundary against observability-as-back-door survived. |
| F37 | per-invite-named-sharing | 2 | yes | included | Rule only: visit invitations are email-based, opt-in per invite, default off, revocable, and no global social surface exists. | Rule only: PLAN §§3.7,4.3,1.2: named email invites, default off, no profiles/follows/public feed. | yes | none | The private-relationship/lending-access rationale was not reconstructed. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | Silent visit log is demand-only; there is no badge count elsewhere, host not told in-session, and no host toast. | PLAN §§4.3,7.5,15.6: host can inspect visit log in settings; no badge, push, email, or attention-driving surface by default. | no | full | The transparency-without-notification rationale survived. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | Rule only: visitor uses the same renderer and sees the host snapshot, minus interactive/host-only fields; show-off rendering is banned. | Rule only: PLAN §§3.7,4.3,1.2: same renderer, visitor snapshot, no show-off/social rendering. | yes | none | The witness-real-birds-not-marketing-rendering why was not reconstructed. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | Idle cadence is 45s/30-60s; narration avoids every micro-shift; high-frequency live-region spam is refused. | PLAN §§9.1,9.5,13.4: idle cadence 45s, event priority, live-region rate checks, no high-frequency spam. | no | full | Slow shared rhythm, screen-reader queue protection, and event-priority exception survived. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | ✓ | ✓ | ✓ |
| F2 | ✓ | ✓ | ✓ |
| F3 | ✓ | ✓ | ✓ |
| F4 | ✓ | ✓ | ✓ |
| F7 | ✓ | ✗ | ✓ |
| F9 | ✓ | ✓ | ✓ |
| F10 | ✓ | ✗ | ✓ |
| F12 | ✓ | ✓ | ✓ |
| F13 | ✓ | ✓ | ✓ |
| F14 | ✓ | ✓ | ✓ |
| F15 | ✓ | ✓ | ✓ |
| F16 | ✓ | ✓ | ✗ |
| F17 | ✓ | ✓ | ✓ |
| F18 | ✓ | ✓ | ✓ |
| F20 | ✓ | ✓ | ✓ |
| F24 | ✓ | ✓ | ✓ |
| F25 | ✓ | ✓ | ✓ |
| F27 | ✓ | ✓ | ✓ |
| F30 | ✓ | ✓ | ✓ |
| F31 | ✓ | ✓ | ✓ |
| F35 | ✗ | ✓ | ✓ |
| F40 | ✓ | ✓ | ✓ |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON `gold_why_totals` |
| Possible total weight | 152 | From score JSON `intent_recovery.total_possible_weight` |
| Reachable gold whys | 49 | S whys always included; F whys included only when feature captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 128.0 / 152.0 | Sum of `weight × recovery-score` over included whys |
| Whys with reconstruction evidence | 49 | Exact reconstruction evidence or rule evidence present in frozen reconstruction |
| Whys with PLAN grounding | 49 | Exact PLAN grounding or rule grounding present |
| `rule_without_why` cases | 7 | What survived without why |
| `plan_only_not_reconstructed` cases | 0 | PLAN carried rationale, reconstruction did not |
| `ungrounded_reconstruction` cases | 0 | Reconstruction asserted rationale not grounded in PLAN |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 46.0 | 85.2% |
| Affective whys | 98.0 | 82.0 | 83.7% |
| Weight 2 whys | 44.0 | 30.0 | 68.2% |
| Weight 3 whys | 108.0 | 98.0 | 90.7% |
| System-level whys | 28.0 | 26.0 | 92.9% |
| Feature-level whys (reachable) | 124.0 | 102.0 | 82.3% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional architecture was slightly stronger: presence, drift, server tick, no-LWW, and privacy boundaries mostly recovered fully. Affective exception whys leaked more often, especially F8, F29, F34, F37, and F39.
- **Weight-3 vs weight-2.** Weight-3 whys survived well because the plan often wrote their layers operationally. The notable partials were F7, F10, F16, and F35, each missing one deeper layer.
- **System-level vs feature-level.** The plan preserved philosophy strongly, and the reconstruction found most of it. Feature-level fidelity dropped when a mechanism was visible but the reason was compressed away.
- **Multi-layer recovery patterns.** Primary implementation layers were usually recovered. Secondary affective layers, such as S2's processed-vs-seen distinction and F7's vector-loss-as-bird-loss layer, were the common misses.
- **Subdomain patterns.** Simulation, sync, privacy, audio, and accessibility were strong. Social sharing and account lifecycle surfaces had more rule-only recoveries.
- **Evidence-bound effects.** Seven feature rows were marked `rule_without_why`; v1-style semantic scoring would likely have awarded more credit for those rules.

What the failure shape suggests: this candidate can plan the whole product and carry most cross-cutting intent, but compresses some headroom whys into build rules. The benchmark is therefore still discriminating at the feature-rationale layer even under complete feature capture.

---

## 4. Recommendations for v2 hardening

- Keep v06 evidence-bound scoring; it is doing useful work by separating capture from rationale recovery.
- Add more feature-level exception whys similar to F29-F40. This run showed the clearest losses where the rule survived but the reason did not.
- Consider splitting evidence-audit counts into rationale evidence vs rule-only evidence, since both are useful but they mean different things.
- Keep multi-layer functional whys: they exposed partial recoveries in F7, F16, and F35 without penalizing the otherwise strong plan too coarsely.
- Leave the system-level three-feature bar in place; it was easy to apply here and clearly separated S2's missing affective layer from its broad cross-cutting preservation.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction was treated as frozen and plan-derived. I did not modify it.
- **Single-run-at-temperature limitation.** This is one run; no variance signal is available inside this score.
- **Borderline capture calls.** None were scored as missed. The closest calls were glossary/prose features, but the plan carried enough implementation detail and vocabulary to build them.
- **System-level cross-cutting.** All S1-S9 met the >=3 inherited-decision bar in PLAN. S2 remained partial only because one layer was not reconstructed.
- **Confabulation cases.** I did not count any ungrounded reconstruction claims; no positive recovery depends on a claim absent from PLAN.
- **Evidence-bound denials.** F5, F8, F19, F29, F34, F37, and F39 were rule-without-why denials.
- **Operational compromise.** TIMING.json had phase 1 and phase 2A timing for runs.001 but no phase 2B timing yet, so the score JSON includes only the bounded available timing fields.

---

End of report.
