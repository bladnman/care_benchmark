# REPORT - CARE run 001

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **48.3%** |
| Intent fidelity | **30.6%** |
| Combined quality | **4764** |

**Diagnostic split:**

- System-level fidelity: **39.3%**
- Feature-level fidelity: **27.5%**

**(Planning, fidelity) coordinate:** `48.3, 30.6`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label | (blank) |
| Timestamp | 2026-05-09T14:42:07Z |
| Candidate model | gemini-3-flash-preview |
| Candidate effort | high |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **58 / 120** = **48.3%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 4 | 66.7% |
| concepts.md | 4 | 2 | 50.0% |
| bird_engine.md | 22 | 14 | 63.6% |
| interactions.md | 20 | 8 | 40.0% |
| aviary_layout.md | 18 | 2 | 11.1% |
| accounts_sync.md | 18 | 9 | 50.0% |
| social_optional.md | 10 | 7 | 70.0% |
| accessibility_perf.md | 18 | 8 | 44.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **58** | **48.3%** |

Per-feature detail:

| Feature # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Scope names Pocket Aviary, birds, aviary, naturalist requirements. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured operationally through server simulation, procedural calls, motion, and load budgets; philosophy label is compressed. |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | No notice-vs-announce rule, greeting surface, or toast refusal beyond generic gamification exclusions. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | no | Naturalist surfaces appear, but the matter-of-fact system exception is absent. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Out-of-scope section covers gamification, custodial mechanics, native apps, and social network features. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Plan starts with 2 birds and caps at 7. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary or domain-term definitions beyond incidental use. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Presence tracking uses visibility + focus + activity and drives drift. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Plan separates personality drift from fast-timescale mood. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | no | Settle is named as an event/gesture but not defined as session end. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Borderline: vector exists and two traits are named, but the full trait set is not enumerated. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Low-pass drift function is explicit. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Borderline: three-week visible target appears; one-week instrumentation target is missing. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Monotonic expressive drift and no negative neglect drift are explicit. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Borderline: fast mood state machine appears, but daily-ish reset is not specified. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Time-of-day, ambient events, and recent interactions drive mood. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Motifs and runtime jitter are specified. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | no | No per-bird call identity or recognizability requirement. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | no | No chorus-system requirement beyond listen-in ducking. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | no | No personality-shaped call timing. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Breathing and scanning micro-motions are specified. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Idle micro-motions are driven by per-mood shaders. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | No species-pool size. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | no | A name field exists, but user assignment and renaming do not. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Borderline: initial adoption of 2 birds is present; auto-selection/catalog refusal is absent. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Cap at 7 is explicit. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Scaling based on aviary age is explicit. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Persistent bird vector plus server-authoritative simulation cover the core mechanism. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Borderline: current_mood is persisted and clients sync snapshots; no neutral-reset refusal. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | No bird-to-bird call/reaction model. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | no | Generic database id is not enough to preserve the identity-continuity rule. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | no | No UI exposure/refusal rule for vector numbers. |
| 33 | Return-greeting on viewer arrival | interactions.md | no | No return-greeting feature. |
| 34 | Greeting variation by absence length | interactions.md | no | No greeting feature or absence-length variation. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | No greeting feature. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | No greeting feature. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit welcome-toast/banner refusal. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in audio re-balancing is explicit. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Other birds are ducked to 0.1, not silenced. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Seed, song, and pool offers are named. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Offers influence mood, but mood/curiosity-specific reactions are absent. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | No offer cooldown. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | no | Settle is named but not tied to session end or evening lighting. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | No close-tab equivalence or opt-in rule. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Auto-generated naturalist Field Notebook appears. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | no | No rarity/noteworthiness rule. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | no | No read-only notebook rule. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Presence pings and validated presence drive drift. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Visibility + focus + activity are explicit. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Streaks and counters are explicitly excluded; days-visited is not separately named. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Borderline: return-from-background resync and server simulation are present; render-pause detail is absent. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No settle undo window. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | no | Single canvas is not the same as one horizontal no-panning scene. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | no | Perches exist but not three proximity zones. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | No bird-choice/user-placement rule. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Day/night overlay is tied to local Date.now(). |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Borderline: golden-hour/night palette appears; quieter calls are absent. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No night-state bird behavior. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | no | Ambient events/static weather appear, not concrete rare weather. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | no | Ambient events may influence mood, but rain/vocal-frequency effect is absent. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | no | No leaf/feather drift. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | no | No parallax. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | no | No chrome/top-bar layout rule. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | No top bar contents. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | No auto-fading top bar. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | no | Initial snapshot/load target appears, but not motion already in progress. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | no | No loading-state visual rule. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | No palette spec. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | No responsive framing/cropping rule. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link auth is explicit. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | No expiry. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Single-user auth and one account-linked aviary model are present. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Account UUID and email_hash are present. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Universal server tick is explicit; cadence is not. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Client requests latest snapshot on load/resync/return from background. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Spring interpolation between Perch A and Perch B is explicit. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Multi-device sync via server-side canonical simulation is explicit. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Borderline: no client authorship and sequential event processing avoid client LWW, but the rule is not named. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Clients never compute drift; server processes events. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | No sync error surface or tone rule. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | no | No per-device token/revocation. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | no | No account export. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | No account deletion. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | no | No telemetry/ML privacy rule. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | no | No aggregate telemetry boundary. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email change flow. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Per-invite opt-in and visitor_email are present. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Per-invite opt-in implies no default public visit surface. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Visitor pulls a read-only snapshot. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Read-only snapshot excludes visitor interactions. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Profiles/chat/public discovery are excluded; avatars not separately named. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | No visit-notification rule. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | no | Invite status/expiry exist, but revocation is not specified. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | no | No visit log. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Borderline: read-only snapshot implies actual state; no show-off refusal. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Public discovery/social network surfaces are excluded. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Naturalist prose narration is explicit. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | no | No narration cadence. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Naturalist sentences are explicit. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Reduced motion uses 2-second cross-fades. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | no | No preserved-charm/non-fallback requirement. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Call captions are present; toggle/mood detail is absent. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | no | No contrast requirement. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Keyboard access to birds/listen-in is present, not all surfaces. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | no | No focus-indicator rule. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Bundle budget is explicit. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | <500ms first-bird target is explicit; device/network context omitted. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | no | 60fps interpolation is present, but not idle motion or 5-year laptop target. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | no | Object pooling addresses GC pauses, not the 30-minute no-growth requirement. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | WebAudio Oscillator/Gain synthesis is client-side. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | no | No WebAudio fallback. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | no | No observability plan. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | no | No tick-latency error budget. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | No browser matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native apps are out of scope; web-only. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification, streaks, levels, and counters are out of scope. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Custodial mechanics are out of scope and birds do not die/get hungry. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Profiles, chat, and public discovery are out of scope. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **39.3%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:13: "Procedural life over static playback" with procedural calls, runtime variation, "breathing" and "scanning". | PLAN.md:88-90: "Birds do not play MP3s; they follow motifs" with runtime variation; 105: procedural breathing/scanning; 28-40: server tick and snapshots. | yes | yes | no | partial | Aliveness mechanisms survive, but the downstream staleness/accompaniment rationale is absent. |
| S2 - notice-never-announce | 4 | included | none | none | no | no | yes | none | Plan excludes gamification but never encodes notice-vs-announce or toast refusal as a product principle. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md:3: "Naturalist product voice, not game pressure" and naturalist Field Notebook/narration. | PLAN.md:13: naturalist "Field Notebook"; 121: "naturalist sentences"; 20: gamification out of scope. | yes | no | no | partial | Naturalist/specific surface survives, but not the full specificity-over-generic-system-states rationale. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md:5: "A calm, bounded aviary" and "small observed place". | PLAN.md:10: "Initial adoption of 2 birds, capping at 7"; 137-140 rollout keeps the same cap. | yes | no | no | partial | Bird-count restraint is visible, but one-screen/no-chrome/palette/audio restraint are mostly missing. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md:3: naturalist product voice carried through notebook and screen-reader narration. | PLAN.md:13: naturalist Field Notebook; 16 and 121: naturalist prose narration/sentences. | yes | no | yes | partial | Naturalist voice survives; the matter-of-fact account/error/accessibility exception does not. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:31: "Presence exists as the signal that drives slow personality change"; 107: "non-punitive absence". | PLAN.md:12: presence tracking as visibility + focus + activity; 79: validated presence changes traits; 80: no signal means no negative drift. | yes | no | no | partial | Presence drives drift, but population-inflation precision and settle/tab-close equivalence are lost. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:7: "Server-authoritative, canonical simulation"; clients "never compute drift" and only report interactions. | PLAN.md:28: server-authoritative simulation; 31: server Universal Tick; 96: clients never compute drift. | yes | yes | no | partial | Server-canonical and multi-device coherence survive; the client-simulation/LWW failure mode is mostly absent. |
| S8 - privacy-first-on-bird-data | 2 | included | none | none | no | no | no | none | The per-bird relationship-data privacy stance and telemetry boundary are not in the plan or reconstruction. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:17: "Accessibility and performance are product constraints"; 47: accessible narration in the same product voice; 49: reduced-motion preserves experience. | PLAN.md:16: accessibility in v1 scope; 107: reduced-motion cross-fades; 119-123: narration, captions, keyboard. | yes | yes | no | partial | Accessibility is in-scope and product-shaped, but the anti-checklist/cost rationale is thin. |

Multi-layer system whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | no | no | no |
| S6 | yes | no | no |
| S7 | yes | yes | no |
| S9 | yes | no | yes |

**Cross-cutting evidence appendix.**

- S1: PLAN has server-authoritative simulation/tick/snapshots, procedural call grammar, idle micro-motion, and first-snapshot performance. Count >=3, but downstream failure rationale is missing.
- S2: PLAN has no gamification and public discovery refusal, but not notice-vs-announce across three surfaces. Count <3.
- S3: PLAN has naturalist Field Notebook, naturalist screen-reader sentences, bird names, and no gamification; the exact specificity rationale is still compressed.
- S4: PLAN has 2-to-7 bird bounds and rollout caps; one-screen/no-chrome/calm palette are not carried. Count <3.
- S5: PLAN carries naturalist voice in notebook and narration, but lacks matter-of-fact account/error/system surfaces. Count <3 for the full split.
- S6: PLAN carries validated presence, drift input, no negative neglect drift, and no gamification, but not laxer-tab inflation or settle/tab-close equivalence.
- S7: PLAN carries server-authoritative simulation, universal tick, state snapshots, no client authorship, and multi-device sync. Count >=3.
- S8: PLAN does not carry the per-bird privacy/telemetry boundary. Count 0.
- S9: PLAN carries v1 accessibility scope, narration, reduced motion, captions, and keyboard access. Count >=3, but anti-checklist rationale is thin.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **27.5%**.

Reachable feature-level whys: **25 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | none | none | yes | none | Rule captured, but B does not recover the precise three-signal rationale or silent population-drift failure. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:71: "damped, gradual personality change"; 109: visible change takes "~3 weeks" and avoids "Tamagotchi" vs "Static". | PLAN.md:37: low-pass filter; 81: visible change takes ~3 weeks; 145: too fast = Tamagotchi, too slow = Static. | no | partial | Low-pass/slowness and failure endpoints survive; one-week-instrument vs three-week-user calibration is missing. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:25: expressive drift through presence without punishing absence; 107: "No signal = 0 change" and birds never become wary. | PLAN.md:78-81: monotonic drift, no signal = 0, birds never become more wary; 145: Tamagotchi risk. | no | partial | Positive/no-negative drift and no-punishment survive; the two-week-return/quieter-not-mistrust consequence is absent. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:29: rejects static playback; "variety and living texture" through runtime jitter. | PLAN.md:88-90: birds do not play MP3s; motifs and runtime pitch/timing jitter. | no | partial | Procedural-not-looped layer survives; chorus artifacts and audio-spine cascade do not. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Per-mood shaders survive as rule, but not the user-reading-mood-without-labels rationale. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | B explicitly says the plan does not explain why 2/5/7 are chosen. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md:83: personality_vector is core simulation state; 7: server-authoritative canonical simulation. | PLAN.md:49: bird personality_vector persisted; 28 and 96: server-authoritative simulation with no client drift authorship. | yes | partial | Server persistence is visible, but the bird-deletion/user-relationship rationale is not. |
| F8 | vector-never-shown-numerically | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F9 | return-greeting | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md:39: notebook carries "naturalist product voice" into journaling. | PLAN.md:13: auto-generated naturalist Field Notebook; 50: notebook entry_text and timestamp. | no | partial | Naturalist notebook prose survives; rarity/read-only/not-feed and stock-log rationale do not. |
| F13 | presence-accounting | 4 | yes | included | none | none | yes | none | The rule is in PLAN, but B does not recover the conjunction or tab-open silent-corruption rationale. |
| F14 | no-streak-counter | 4 | yes | included | none | none | yes | none | Streaks/counters are excluded, but the presence-for-birds vs counter rationale is absent. |
| F15 | scene-loads-with-motion | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F16 | synthetic-account-id | 4 | yes | included | none | none | yes | none | UUID/email_hash rule is present, but B says the rationale for email_hash/settings is not recoverable; PII-leak rationale absent. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md:61: server advances aviary state; 63: tick collects events, evaluates drift, transitions moods, snapshots, broadcasts. | PLAN.md:31: Universal Tick advances state; 35-40: tick pipeline; 96: clients never compute drift. | no | partial | Server tick and client-snapshot model survive; the two-client divergent-simulation failure is absent. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md:119: clients "never compute drift" and only report interactions; 97: append-only event reporting. | PLAN.md:67: events are appended; 96: clients never compute drift; 143: server processes the event log sequentially. | yes | partial | No-client-authorship mechanism survives; the lost-drift LWW failure rationale is not articulated. |
| F19 | sync-conflict-tone | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F20 | no-per-bird-ml-telemetry | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md:45: visits are allowed "without creating a social network or allowing visitor authorship." | PLAN.md:15: read-only visit invitations; 71: visitor pulls read-only snapshot. | no | full | Recovered as observation/read-only rather than visitor authorship/co-presence. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | Public discovery/social network surfaces are refused, but comparison/public-surface rationale is absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:47: accessible narration in "the same product voice"; 145: latest state in naturalist prose. | PLAN.md:121: StateSnapshot converted into naturalist sentences in aria-live region. | no | partial | Naturalist running-prose/voice-continuity survives; ARIA-label automation warning is absent. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md:49: reduced motion preserves the experience while disabling path animations. | PLAN.md:107: path animations disabled and replaced with 2-second cross-fades. | no | partial | Cross-fade rendering survives; continuing calls/mood/notebook and stripped-fallback consequence are absent. |
| F26 | ttfb-500ms | 2 | yes | included | none | none | yes | none | The performance target survives, but not the felt-aliveness threshold rationale. |
| F27 | no-gamification-non-goal | 4 | yes | included | none | none | yes | none | No-gamification rule survives, but not the counter/engagement-foothold rationale. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md:107: "non-punitive absence"; 55: custodial mechanics are out of scope. | PLAN.md:21: birds do not die or get hungry; 80: no signal = 0 and birds never become more wary; 145: Tamagotchi risk. | no | full | Recovered as refusal of punitive/custodial absence mechanics. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Two starter birds are present, but B says the why for the starter/count choice is not recoverable. |
| F30 | age-based-bird-offers | 4 | yes | included | none | none | yes | none | Age-based scaling appears, but B explicitly says the why age should control scaling is absent. |
| F31 | stable-bird-identity | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | Persisted current_mood/snapshots survive, but not the no-neutral-reset/continued-while-away rationale. |
| F33 | notebook-read-only-observer-record | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F34 | account-export-relationship-copy | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F36 | aggregate-telemetry-boundary | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Per-invite named sharing survives as rule, but not the private-relationship/control rationale. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Read-only snapshot implies actual state, but no show-off/marketing-rendering rationale is recovered. |
| F40 | narration-cadence-slow | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | no | no | no |
| F2 | yes | no | yes |
| F3 | yes | yes | no |
| F4 | yes | no | no |
| F7 | yes | no | no |
| F9 | unreachable | unreachable | unreachable |
| F10 | unreachable | unreachable | unreachable |
| F12 | yes | no | no |
| F13 | no | no | no |
| F14 | no | no | no |
| F15 | unreachable | unreachable | unreachable |
| F16 | no | no | no |
| F17 | yes | yes | no |
| F18 | yes | no | yes |
| F20 | unreachable | unreachable | unreachable |
| F24 | yes | yes | no |
| F25 | yes | no | no |
| F27 | no | no | no |
| F30 | no | no | no |
| F31 | unreachable | unreachable | unreachable |
| F35 | unreachable | unreachable | unreachable |
| F40 | unreachable | unreachable | unreachable |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | From score JSON intent_recovery.total_possible_weight |
| Reachable gold whys | 34 | S whys always included; F whys only when captured |
| Excluded unreachable feature whys | 15 | Denominator exclusions |
| Recovered / reachable weight | 33 / 108 | Weighted numerator over included whys |
| Whys with reconstruction evidence | 18 | Rationale evidence, not mere rule mentions |
| Whys with PLAN grounding | 18 | Grounding for the same rationale |
| rule_without_why cases | 18 | Mechanism survived without why |
| plan_only_not_reconstructed cases | 0 | No clear plan-only rationale rows |
| ungrounded_reconstruction cases | 0 | No scoring rows required confabulation penalty beyond missing grounding |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 42 | 12 | 28.6% |
| Affective whys | 66 | 21 | 31.8% |
| Weight 2 whys | 28 | 7 | 25.0% |
| Weight 3 whys | 80 | 26 | 32.5% |
| System-level whys | 28 | 11 | 39.3% |
| Feature-level whys (reachable) | 80 | 22 | 27.5% |

## 3. Diagnostic patterns

- **Affective vs functional.** Affective whys recovered 21/66 (31.8%), slightly above functional recovery at 12/42 (28.6%). The plan's strongest affective carry-through is non-punitive drift/accessibility; weaker areas include notice-never-announce and social/privacy nuance.
- **Weight-3 vs weight-2.** Higher-weight whys recovered 26/80; weight-2 whys recovered 7/28. Multi-layer rows often got one or two primary implementation layers but dropped secondary/downstream rationale.
- **System-level vs feature-level.** System-level fidelity (39.3%) is meaningfully better than feature-level fidelity (27.5%). The planner preserved broad architecture more than feature-specific exceptions.
- **Subdomain patterns.** Server sync, drift, procedural audio, and accessibility have usable mechanisms. Return-greeting, layout/chrome, privacy/telemetry, account export/deletion, and visit-log details are mostly absent.
- **Evidence-bound effects.** Many rows were rule-only: F14, F16, F23, F26, F27, F30, F37, and F39 are clear examples where a builder could implement the rule but not understand the product reason.

## 4. Recommendations for v2 hardening

- Keep the v06 evidence-bound ledger: it exposed a large rule-without-why band that a simpler semantic scorer would likely over-credit.
- Add more targeted headroom around social/privacy/account settings. This run captured many social mechanics but lost notification, visit-log, private-relationship, and telemetry rationale.
- Preserve multi-layer whys for calibration features. F2, F17, and F18 show that candidates often recover primary architecture while dropping the downstream failure mode.
- Consider adding an explicit capture sub-score for named exclusions. Generic no-gamification captured several rules but did not reliably capture no-toast/no-notification/no-leaderboard variants.

## 5. Methodology caveats

- **Fresh-context fidelity.** I treated the reconstruction as frozen and scored only the allowed phase-two, plan, metadata, timing, and reconstruction files.
- **Single-run limitation.** This is one candidate plan and one reconstruction; no variance signal is available inside this slot.
- **Borderline capture calls.** I leaned inclusive on several compact mentions: personality vector traits, three-week calibration, two-starter adoption, mood persistence, background return, and actual visitor snapshot. These affect planning quality more than fidelity.
- **System cross-cutting.** The strict >=3-inheritance bar helped S7 and S9, but limited S2, S4, and S5 where only one or two surfaces carried the principle.
- **Confabulation.** I did not find a row where recovery credit depended on an ungrounded reconstruction assertion; thin inferences were scored as none or partial.
- **Evidence-bound denials.** Most none rows were denied because the reconstruction/plan preserved a rule without the gold rationale, not because the feature was unreachable.
