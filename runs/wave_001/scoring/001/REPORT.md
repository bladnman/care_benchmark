# REPORT - CARE run 001

Variant v06 evidence-bound clean scoring for Pocket Aviary. The frozen reconstruction was read-only; feature-level recovery was credited only where both reconstruction evidence and PLAN grounding were available.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **92.5%** |
| Intent fidelity | **55.5%** |
| Combined quality | **9205** |

**Diagnostic split:**

- System-level fidelity: **75.0%**
- Feature-level fidelity: **50.8%**

**(Planning, fidelity) coordinate:** `(92.5, 55.5)` - plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-06T15:20:49Z |
| Candidate model | gemini-3.8-flash |
| Candidate effort | medium |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **111 / 120** = **92.5%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 16 | 80.0% |
| aviary_layout.md | 18 | 16 | 88.9% |
| accounts_sync.md | 18 | 16 | 88.9% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **111** | **92.5%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured by the executive summary: tranquil browser-based virtual aviary and observational relationship. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured through continuing-without-user, no-spinner first paint, procedural calls, and alive/quiet/lasting conclusion. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | borderline: compressed into birds noticing the user plus anti-gamification/no-push rules; no exact principle phrase. |
| 4 | Voice-and-tone guide for product surface | product_brief.md | yes | Exact naturalist vs matter-of-fact voice partition in PLAN 1.3. |
| 5 | "What this is not" callout | product_brief.md | yes | PLAN 1.2 prohibits native apps, gamification, Tamagotchi mechanics, social networking, and engagement notifications. |
| 6 | Restraint-over-richness scope statement | product_brief.md | yes | Start with two birds, maximum seven, single horizontal scene, calm naturalist palette. |
| 7 | Glossary of domain terms | concepts.md | yes | borderline: no glossary section, but core terms are operationally defined across mood, personality, call, settle, and presence sections. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Strict conjunction of visible document, window focus, and recent pointer/key activity. |
| 9 | Definition of personality vector vs mood | concepts.md | yes | PLAN separates hidden slow personality vector from fast-timescale mood state. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Settle gesture is a soft session-end gesture with evening lighting and quieter calls. |
| 11 | Personality vector traits | bird_engine.md | yes | PLAN enumerates boldness, social warmth, vocal frequency, plumage saturation, and curiosity. |
| 12 | Personality drift function | bird_engine.md | yes | Low-pass drift filter over presence/listen/offer/ambient inputs. |
| 13 | Drift rate calibration | bird_engine.md | yes | PLAN gives session, one-week instrument, and three-week user-perception targets. |
| 14 | Personality drift is monotonic toward expressive | bird_engine.md | yes | Traits increase on positive inputs and never decrease from absence or neglect. |
| 15 | Mood state | bird_engine.md | yes | PLAN stores wary, content, curious, drowsy, and alert mood states. |
| 16 | Mood inputs | bird_engine.md | yes | Mood transitions use local time, weather, recent interactions, and social contagion. |
| 17 | Procedural call grammar | bird_engine.md | yes | Species motif grammars combine and vary motifs via WebAudio. |
| 18 | Per-bird call signature | bird_engine.md | yes | Per-bird call seeds and recognizable procedural call signatures are included. |
| 19 | Chorus mixing | bird_engine.md | yes | Chorus scheduler, stochastic staggering, spatial positioning, and no stacked loops are included. |
| 20 | Call timing shaped by personality | bird_engine.md | yes | borderline: vocal-frequency trait and vocal_frequency_tier are present; timing use is less explicit than other call rules. |
| 21 | Idle micro-motion | bird_engine.md | yes | Saccadic eye/head movement, respiratory sway, preening, and weight shuffle are specified. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Content, wary, drowsy, alert, and curious states map to visible behaviors and perches. |
| 23 | Bird species pool for v1 | bird_engine.md | yes | Six species/archetypes are enumerated. |
| 24 | Bird naming | bird_engine.md | yes | Naming flow at onboarding and settings is in scope. |
| 25 | Adoption flow | bird_engine.md | yes | Exactly two starter birds are randomly selected at account creation. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Aviary population is capped at seven birds. |
| 27 | Adding a third+ bird | bird_engine.md | yes | New birds unlock strictly by aviary age milestones. |
| 28 | Personality vector persistence | bird_engine.md | yes | Personality vectors are server-canonical, persisted, and updated by the server tick. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Mood is server-canonical and clients pull snapshots rather than resetting local mood. |
| 30 | Bird-to-bird interaction | bird_engine.md | yes | Social contagion and alarm response between neighboring birds are specified. |
| 31 | Bird identity stability | bird_engine.md | yes | borderline: stable bird UUID separate from name/species is present; migration/replacement language is not explicit. |
| 32 | Personality vector exposure | bird_engine.md | yes | Personality vectors are hidden and stripped from client snapshots. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | One bird greets within one to two seconds of return. |
| 34 | Greeting variation by absence length | interactions.md | yes | Return-greeting is modulated by absence duration. |
| 35 | Greeting variation by bird boldness | interactions.md | yes | Return-greeting is modulated by boldness. |
| 36 | Greeting stagger | interactions.md | yes | Snapshot includes stagger delay and chorus scheduler avoids synchronized calls. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit prohibition on textual welcome toasts, banners, modals, or return copy. |
| 38 | Listen-in interaction | interactions.md | yes | Pointer/keyboard focus raises one bird in the audio mix. |
| 39 | Listen-in mix decay | interactions.md | yes | Focused bird rises while ambient birds attenuate without full mute. |
| 40 | Offer interaction | interactions.md | yes | Seed, song fragment, and still pool offer affordances are included. |
| 41 | Offer reaction varies by mood and curiosity | interactions.md | yes | Offer reactions are shaped by curiosity and mood. |
| 42 | Offer cooldown | interactions.md | yes | Per-bird offer cooldown is three minutes. |
| 43 | Settle gesture | interactions.md | yes | Settle shifts lighting to evening and quiets calls. |
| 44 | Settle is opt-in | interactions.md | no | No explicit closing-tab-is-valid or engine-equivalent-to-settle rule. |
| 45 | Field notebook auto-entries | interactions.md | yes | Server-generated sparse naturalist observation log. |
| 46 | Field notebook entry frequency | interactions.md | yes | Rate limited to roughly every 48-72 hours or 2-4 days. |
| 47 | Field notebook is read-only | interactions.md | yes | PLAN states the notebook is read-only. |
| 48 | Presence accounting | interactions.md | yes | Presence events are recorded using strict active-attention criteria. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | All three signals are required simultaneously. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | No streaks, visit counters, green-dot calendars, or engagement habit surfaces. |
| 51 | Background-tab pause | interactions.md | no | Server continuation and visibility polling are present, but client render-pause-when-hidden is not specified. |
| 52 | Click-anywhere-to-undo settle | interactions.md | no | PLAN has a five-second undo grace window, but not the click-anywhere undo affordance. |
| 53 | Single horizontal scene | aviary_layout.md | yes | Single canvas and no panning/no scrolling are specified. |
| 54 | Three perch zones | aviary_layout.md | yes | Front, middle, and back perch zones are specified. |
| 55 | Bird-chosen perch | aviary_layout.md | yes | borderline: mood/personality drive perch use; no explicit user-cannot-place-birds sentence. |
| 56 | Day/night cycle tied to local time | aviary_layout.md | yes | Lighting and mood use local time and timezone. |
| 57 | Evening palette shift | aviary_layout.md | yes | Settle transitions to evening lighting and day/night rendering changes sky/horizon. |
| 58 | Night state | aviary_layout.md | yes | borderline: night shifts most birds drowsy and includes a nightjar archetype, but not the exact PRD night-state sentence. |
| 59 | Ambient weather | aviary_layout.md | yes | Rare rain and wind are in scope. |
| 60 | Weather affects mood | aviary_layout.md | yes | Rain increases wary/drowsy probability and reduces vocalization. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Ambient particles include leaf and feather drift. |
| 62 | Foreground/background parallax | aviary_layout.md | yes | Subtle parallax layer is specified. |
| 63 | No UI chrome inside aviary view | aviary_layout.md | yes | Actions live in a top bar and the scene is a single canvas. |
| 64 | Top bar contents | aviary_layout.md | yes | Account/settings/accessibility/notebook/offer affordances are included across scope and keyboard model. |
| 65 | Top bar auto-fades when cursor idle | aviary_layout.md | no | No top-bar auto-fade behavior is specified. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | First frame paints birds mid-breath, mid-preen, or mid-scan. |
| 67 | Loading state is quiet field | aviary_layout.md | yes | Quiet sky gradient and soft breeze replace spinner/skeleton/progress. |
| 68 | Empty-aviary state | aviary_layout.md | no | No between-adoption-and-first-bird empty aviary state is specified. |
| 69 | Color palette spec | aviary_layout.md | yes | Calm naturalist palette and dynamic sky/horizon colors are specified. |
| 70 | Responsive scene never crops bird | aviary_layout.md | yes | Letterboxing/branch compression keep all birds and perches visible. |
| 71 | Email + magic-link sign-in | accounts_sync.md | yes | Email magic link authentication is in scope. |
| 72 | Magic link expiry | accounts_sync.md | yes | Magic links have a 15-minute token TTL. |
| 73 | Single-user accounts | accounts_sync.md | yes | Single aviary per account is specified. |
| 74 | Synthetic account ID | accounts_sync.md | yes | Synthetic UUIDv4 account_id is required. |
| 75 | Server-side simulation tick | accounts_sync.md | yes | Server worker ticks every roughly 60 seconds. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Clients pull snapshots on initial load, visibility return, refocus, and keepalive. |
| 77 | Client interpolates between snapshots | accounts_sync.md | yes | Client interpolates snapshots for smooth motion. |
| 78 | Multi-device sync | accounts_sync.md | yes | Multi-device read consistency via server-canonical state is specified. |
| 79 | Last-write-wins forbidden | accounts_sync.md | yes | Client-side LWW for personality data is rejected. |
| 80 | Conflict resolution: server tick only writer | accounts_sync.md | yes | Only server simulation worker writes personalities and moods. |
| 81 | Sync conflict surface tone | accounts_sync.md | yes | Errors and sync/account surfaces use matter-of-fact copy. |
| 82 | Per-device session token | accounts_sync.md | yes | Session token generation and revocation are in scope. |
| 83 | Account export | accounts_sync.md | yes | borderline: export endpoint and transactional export exist; exact JSON contents are not enumerated. |
| 84 | Account deletion | accounts_sync.md | yes | borderline: 30-day restore/scheduled deletion is present; hard-delete cascade is implied more than specified. |
| 85 | No per-bird ML telemetry | accounts_sync.md | yes | Telemetry excludes per-bird interactions and ML-style relationship data. |
| 86 | Aggregate-only telemetry | accounts_sync.md | yes | Only aggregate operational telemetry is collected. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link in account settings is specified. |
| 88 | Email change flow | accounts_sync.md | no | No verify-new-address email change flow is specified. |
| 89 | Visit invitations | social_optional.md | yes | Visits are opt-in and shared by revocable email magic link. |
| 90 | Visits default OFF | social_optional.md | yes | Default disabled visits are specified. |
| 91 | Visit read-only ambient view | social_optional.md | yes | Visits are read-only ambient views. |
| 92 | Visitor cannot trigger interactions | social_optional.md | yes | Visitor actions are disabled and POST /events is blocked. |
| 93 | No chat/comments/avatars during visits | social_optional.md | yes | No social network mechanics include comments, profiles, avatars, and real-time co-presence. |
| 94 | No friend-visited notification by default | social_optional.md | yes | borderline: no push/engagement notifications and visit_notifications_enabled default false cover the rule. |
| 95 | Visit revocation | social_optional.md | yes | Invite revocation endpoint and revocable links are specified. |
| 96 | Visit log | social_optional.md | yes | Visit logs record who/when via invitation records. |
| 97 | Visitor sees host aviary as-is | social_optional.md | yes | Visit payload is identical to owner snapshot with actions disabled. |
| 98 | No leaderboards/discovery/public aviaries | social_optional.md | yes | No leaderboards, discovery feeds, public aviaries, or profiles. |
| 99 | Screen-reader narration running prose | accessibility_perf.md | yes | Naturalist aria-live narration is specified. |
| 100 | Narration cadence is slow | accessibility_perf.md | yes | Idle narration every 30-45 seconds with polite live region. |
| 101 | Narration prose naturalist, not announcement-style | accessibility_perf.md | yes | Narration is a naturalist accompaniment rather than mechanical state changes. |
| 102 | Reduced-motion mode | accessibility_perf.md | yes | Reduced motion replaces movement with slow cross-fades. |
| 103 | Reduced-motion charm preserved | accessibility_perf.md | yes | Reduced motion is an intentional high-craft alternate aesthetic. |
| 104 | Captioning toggle for procedural calls | accessibility_perf.md | yes | borderline: call captions and accessibility settings exist; the explicit toggle is not named. |
| 105 | WCAG AA contrast | accessibility_perf.md | yes | Contrast is enforced at 4.5:1. |
| 106 | Keyboard-only navigation | accessibility_perf.md | yes | Full keyboard navigation model is specified. |
| 107 | Focus indicators visible | accessibility_perf.md | yes | High-contrast focus ring is specified. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | 2MB gzipped bundle ceiling with analyzer gate. |
| 109 | Time to first bird visible <500ms | accessibility_perf.md | yes | Under-500ms first-bird target on 4G mid-tier mobile. |
| 110 | 60fps idle motion target | accessibility_perf.md | yes | 60fps sustained runtime budget. |
| 111 | No memory growth over 30 minutes | accessibility_perf.md | yes | Net-zero heap growth soak test. |
| 112 | Procedural audio client-side | accessibility_perf.md | yes | WebAudio procedural synthesis; no large audio downloads. |
| 113 | Audio fallback | accessibility_perf.md | yes | Graceful silence with captions; no recorded fallback. |
| 114 | Performance observability | accessibility_perf.md | yes | Synthetic/RUM and aggregate server metrics are specified. |
| 115 | Simulation tick latency error budget | accessibility_perf.md | yes | p99 under five seconds with Prometheus alerting. |
| 116 | Browser support matrix | accessibility_perf.md | no | No last-two-major browser support matrix is specified. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Web-only; no iOS/Android native packages or wrappers. |
| 118 | Out of scope: gamification | non_goals.md | yes | No achievements, streaks, scores, badges, XP, ranks, or tiers. |
| 119 | Out of scope: Tamagotchi-style mechanics | non_goals.md | yes | No death, hunger, distress, decaying happiness, or custodial chores. |
| 120 | Out of scope: social network surfaces | non_goals.md | yes | No profiles, follows, public feed, comments, leaderboards, or social graph. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **75.0%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | R system 5: "feel as if it has been continuing without the user"; R Audio: "forbids recorded loops" | PLAN 7.2: "core principle that the aviary has been continuing without the user"; PLAN 8.1 procedural calls; PLAN 7.3 micro-motion | yes | yes | no | partial | Continuing/aliveness and cross-layer mechanics survived, but the downstream staleness/leak-across consequence was not reconstructed. |
| S2 - notice-never-announce | 4 | included | none | PLAN 1.2 no gamification/no push; PLAN 5.3 age-based adoption; PLAN 1.1 return greeting by one bird | no | yes | no | partial | The PLAN repeatedly refuses engagement loops, but B did not recover the announce-versus-notice affective rationale. |
| S3 - charm-from-specificity | 2 | included | R system 8: "strictly lowercase, present-tense, bird-named, observational, evocative"; R Field Notebook: "unique moments" | PLAN 1.3 naturalist voice; PLAN 5.4 noteworthy moments; PLAN 9.2 captions derived from call shape | yes | yes | no | full | Specific, naturalist, bird-named surfaces were preserved across notebook, narration, captions, and visible behavior. |
| S4 - restraint-over-richness | 2 | included | none | PLAN 1.1 two starters/max seven; PLAN 7.1 no panning; PLAN 8.3 no full mute; PLAN 10.1 bundle/TTFBird budgets | no | yes | no | partial | The PLAN preserves restraint cross-cuttingly, but B did not identify the restraint-over-richness principle as system intent. |
| S5 - naturalist-voice-with-system-exception | 2 | included | R system 8: "Product voice is split into two mutually exclusive registers" | PLAN 1.3 naturalist voice vs matter-of-fact voice; PLAN 4.1 errors; PLAN 9.1 narration | yes | yes | no | full | The register split and system-surface exception were explicitly reconstructed and grounded. |
| S6 - presence-is-real-interaction | 4 | included | R Presence Accounting: "strict conjunction of visible document, focused window, and recent pointer or key activity" and "guards against drift inflation" | PLAN 1.1 strict three-signal presence; PLAN 5.1 presence as drift input; PLAN 6.2 bounded presence deduplication | yes | yes | no | partial | Precision and drift-inflation rationale survived; settle/tab-close equivalence did not. |
| S7 - simulation-runs-server-side | 4 | included | R system 6: "server-canonical and client-projected"; R Sync: "rejects client-side simulation... last-write-wins" | PLAN 2.2 client is projection/server is canonical; PLAN 6.1 single canonical server; PLAN 12 multi-device overwrite risk | yes | yes | no | full | Server tick, multi-device coherence, and rejection of client/LWW failure modes were all recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | R system 7: "Privacy is designed as an airgap"; R Telemetry: "lived aviary details out of telemetry" | PLAN 3.1 synthetic ID/PII airgap; PLAN 3.3 telemetry isolation; PLAN 10.3 privacy tests | yes | yes | no | full | The technical privacy boundary was reconstructed as architecture, not policy-only. |
| S9 - accessibility-as-first-class-surface | 4 | included | R system 9: "Accessibility is a primary aesthetic surface"; "fully realized naturalist accompaniment"; "intentional, high-craft alternate aesthetic" | PLAN 1.1 accessibility in v1 scope; PLAN 9.1 narration; PLAN 7.4 reduced motion; PLAN 9.3 keyboard; PLAN 9.4 contrast | yes | yes | no | full | B recovered the charm-not-checklist framing and v1 first-class treatment. |

For multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | no | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: return-greeting, procedural WebAudio, zero-spinner first frame/quiet field, micro-motion, reduced motion, TTFBird.
- S2: no gamification, no push/engagement notifications, age-based bird arrivals, no social network mechanics; no explicit welcome-toast ban.
- S3: naturalist notebook, screen-reader prose, call captions, named birds, hidden personality values, no public comparison surfaces.
- S4: two starters/max seven, single no-panning scene, no UI-heavy soloable audio mixer, lean bundle/fast first paint.
- S5: naturalist product voice for notebook/narration/captions, matter-of-fact auth/error/account/accessibility/settings surfaces.
- S6: strict presence monitor, drift input weighting, monotonic no-punishment drift, presence deduplication, no streak counters.
- S7: server tick, client projection, append-only interactions, multi-device snapshots, no LWW, no client trait mutation.
- S8: synthetic account IDs, encrypted email isolation, telemetry airgap, aggregate metrics, export/deletion account surfaces, no social/public metrics.
- S9: naturalist screen-reader narration, reduced-motion alternate aesthetic, captions, keyboard navigation, visible focus, WCAG contrast, included in v1 scope.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **50.8%**.

Reachable feature-level whys: **38 / 40**. F10 and F11 were unreachable because their anchor features were not captured.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | R Presence Accounting: "strict conjunction of visible document, focused window, and recent pointer or key activity"; "guards against drift inflation" | PLAN 1.1 strict conjunction; PLAN 5.1 presence-time input; PLAN 6.2 interval union prevents drift inflation | no | partial | Recovered precision and drift inflation; missed silent load-bearing failure/user-expectation consequence. |
| F2 | drift-function | 4 | yes | included | R Drift: "very slow" low-pass filter; R harness: one-week and three-week calibration targets | PLAN 5.1 low-pass filter; PLAN 5.1 one-week and three-week targets | no | partial | Low-pass and calibration survived; Tamagotchi-vs-screensaver failure band did not fully survive. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | R system 4: "Absence must never become punishment"; R Neglect: "do not regress" | PLAN 1.2 neglect produces ambient quietness; PLAN 5.1 never decrease; PLAN 11.2 six-month neglect harness | no | full | The no-punishment/no-Tamagotchi exception was recovered with the ambient-quietness consequence. |
| F4 | procedural-call-grammar | 4 | yes | included | R Audio Pipeline: "forbids recorded loops"; static samples "violate the bundle budget and destroy chorus dynamics" | PLAN 8.1 synthesized via WebAudio; PLAN 8.1 no static samples; PLAN 8.2 chorus staggering | no | partial | Recovered chorus/bundle cascade, not the audible-dead-software/spell-break reason. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Mood-shaped motion is planned, but the why that mood must be legible without labels is not recovered. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | The cap appears, but the empirical recognizability/per-bird relationship rationale is absent. |
| F7 | personality-vector-persistence | 4 | yes | included | R Hidden Personality Vectors: "Server Canonical Only, Never Exposed to Client"; R Client is Projection/Server is Canonical | PLAN 2.2 server canonical; PLAN 3.2 bird_personalities table; PLAN 6.1 no client personality writes | no | partial | Canonical persistence and sync cascade survived; deleting-the-bird relationship rationale did not. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | none | yes | none | The hidden-vector rule survived, but not the anti-stat-management why. |
| F9 | return-greeting | 4 | yes | included | R Return-Greeting: "the birds notice the user"; one bird within 1-2 seconds; modulated by absence, boldness, and mood | PLAN 1.1 return-greeting; PLAN 4.2 return_greeting payload; PLAN 8.2 stochastic staggering | no | partial | Greeting mechanics and variation survived; no-toast/canned-cue downstream consequence did not. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured in PLAN; excluded from fidelity denominator. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Closing-tab equivalence was not captured, so the why is unreachable. |
| F12 | field-notebook-prose | 4 | yes | included | R Field Notebook: "unique moments"; "sparse intervals"; "lowercase present-tense naturalist prose instead of engagement feedback" | PLAN 1.1 read-only naturalist log; PLAN 5.4 noteworthy moments; PLAN 5.4 rate limit and style invariant | no | full | Naturalist prose, non-event-log function, rarity, and read-only character were recovered. |
| F13 | presence-accounting | 4 | yes | included | R Presence Accounting: strict conjunction and dominant drift weight; "guards against drift inflation" | PLAN 1.1 strict three signals; PLAN 5.1 presence input; PLAN 6.2 interval union deduplication | no | partial | Precision survived; silent population-wide failure details did not. |
| F14 | no-streak-counter | 4 | yes | included | R No Gamification: "must never measure or reflect back user engagement habits" | PLAN 1.2 zero streaks/visit counters/green-dot calendars; PLAN 12 accidental streak gamification risk | no | partial | The engagement-counter refusal survived, but the intention-rotation and adjacent-disguise layers did not. |
| F15 | scene-loads-with-motion | 4 | yes | included | R Zero-Spinner: uphold continuing-without-user; R First-Frame: birds begin mid-breath/mid-preen/mid-scan; R Quiet Field | PLAN 7.2 inlined snapshot, first-frame mid-action, quiet field; PLAN 2.2 server snapshots | no | full | First-frame motion, server snapshot logic, and quiet field no-spinner consequence were recovered. |
| F16 | synthetic-account-id | 4 | yes | included | R Synthetic Account ID: schemas, messaging, cache keys, partition tokens, and telemetry use synthetic UUIDs; email encrypted and isolated | PLAN 3.1 synthetic account_id; PLAN 3.2 encrypted email and hash; PLAN 2.1 PII scrubbing | no | partial | PII isolation survived; the impossible-to-retrofit design-time consequence did not. |
| F17 | server-side-simulation-tick | 4 | yes | included | R Simulation tick: "persistent heart"; R Sync: server remains sole writer and clients pull snapshots | PLAN 2.1 simulation worker; PLAN 5 tick loop; PLAN 6.1 single canonical server | no | partial | Server tick and sync coherence survived; divergent-client merge failure was only implied. |
| F18 | no-last-write-wins-personality | 4 | yes | included | R Single Canonical Server rejects client-side LWW; R No Absolute State: attempts to send trait values are rejected | PLAN 6.1 rejects LWW; PLAN 6.2 no absolute state from client; PLAN 12 multi-device overwrite mitigation | no | partial | Implementation rule survived; concrete invisible-overwrite failure scenario did not. |
| F19 | sync-conflict-tone | 2 | yes | included | R Matter-of-Fact Voice: operational copy is clear, helpful, neutral, and free of false warmth or simulated naturalist charm | PLAN 1.3 matter-of-fact voice; PLAN 4.1 matter-of-fact error copy | no | full | B recovered the system-clarity exception to naturalist tone. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | R Privacy airgap; R Telemetry Complete Airgap; R Excluded Telemetry excludes traits, presence duration, and offer frequencies | PLAN 3.3 prohibits telemetry from interaction/personality/notebook; PLAN 10.3 privacy tests | no | partial | Technical boundary survived, but private-relationship-converted-to-data-product rationale was not explicit. |
| F21 | visit-read-only-ambient | 2 | yes | included | R Social Affordance: read-only, no co-presence, no visitor interaction, and no presence or drift recorded | PLAN 1.1 zero co-presence/zero drift; PLAN 4.3 actions disabled and POST /events blocked | no | full | Observation-not-co-presence rationale was recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | none | yes | none | Default-off notification mechanics are present, but the attention-driver/social-loop rationale is absent. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | R No Social Network Mechanics: excludes discovery feeds, profiles, comments, leaderboards, follower graphs, and co-presence to keep visits from becoming a social network | PLAN 1.2 no social network mechanics; PLAN 1.2 no leaderboards/discovery/public feeds | no | full | The different-product/social-network rationale survived, though not in the exact comparison language. |
| F24 | sr-narration-running-prose | 4 | yes | included | R Screen-Reader Narration: "fully realized naturalist accompaniment rather than mechanical state change announcements" | PLAN 9.1 naturalist aria-live narration; PLAN 9.1 idle and interaction prose; PLAN 1.1 first-class accessibility | no | full | Running prose, same-product accessibility, and anti-ARIA-automation rationale survived. |
| F25 | reduced-motion-mode | 4 | yes | included | R Reduced-Motion Mode: "intentional, high-craft alternate aesthetic"; cross-fading still poses and removed particles honor reduced motion | PLAN 7.4 reduced-motion cross-fades; PLAN 1.1 accessibility first-class; PLAN 9 inclusion surfaces | no | partial | Alternate-rendering and not-lesser-mode survived; calls/notebook/drift continuity layer did not. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | R TTFBird: under-500ms target matches "instant-start intent" | PLAN 10.1 <500ms on 4G mid-tier mobile; PLAN 10.2 inline initial state and synchronous first frame | no | full | The affective-performance bridge was recovered as instant-start intent. |
| F27 | no-gamification | 4 | yes | included | R No Gamification: "must never measure or reflect back user engagement habits" | PLAN 1.2 zero gamification elements; PLAN 12 accidental streak gamification risk | no | partial | The rule and engagement-habit reason survived; the predictable temptation and foothold cascade did not. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | R No Tamagotchi: avoid distress, chores, and negative regression; "Neglect produces only ambient quietness" | PLAN 1.2 no hunger/death/distress; PLAN 5.1 neglect delta zero | no | full | The observational-not-custodial refusal and no-punishment rationale survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Auto-selected starter birds are planned, but the arrivals-not-catalog/selection-optimization why is absent. |
| F30 | age-based-new-bird-offers | 4 | yes | included | R Bird Population Mechanics: "relationships deepened over time rather than unlocked achievements"; no clicks/purchases/engagement acceleration | PLAN 5.3 age-based adoption pacing; PLAN 1.1 paced by chronological age; PLAN 5.3 no acceleration | no | partial | Age/deepening and reward-loop refusal survived; economy/erosion downstream layer did not. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | A stable bird UUID is present in the PLAN, but B did not recover identity-continuity rationale. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | Server-canonical mood is planned, but no neutral-reset/continuity illusion why was recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only notebook rule is present, but the observer-record-not-user-journal why is absent. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export endpoint exists, but the user-owns-relationship-copy/quiet-QOL why is absent. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | Deletion mechanics are planned, but regret, privacy, and no-residue rationale layers are not reconstructed. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | R Metrics Collected/Excluded: observability stays operational; R Privacy Enforcement keeps telemetry from relationship fields | PLAN 3.3 aggregate metrics only; PLAN 10.3 no account_id/bird_id/name/personality/interaction history | no | full | The technical observability boundary was recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | R Social Affordance: opt-in ambient share via revocable email magic link, default disabled, no public social mechanics | PLAN 1.1 opt-in read-only visits; PLAN 4.3 invite by visitor_email; PLAN 1.2 no discovery/social network | no | full | Per-invite, host-controlled, non-discoverable sharing rationale survived at a semantic level. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | Visit log exists, but on-demand transparency versus notification-surface rationale is absent. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Identical snapshot output is planned, but no-show-off/anti-marketing rationale is absent. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | R Screen-reader idle throttle: every 30-45 seconds, polite live region, prevents queue overload and avoids interrupting | PLAN 9.1 idle every 30-45s; PLAN 9.1 aria-live polite; PLAN 1.1 naturalist narration | no | partial | Queue-overload and sparse observational pacing survived; same-rhythm-as-visual layer did not. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | no | yes | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | no |
| F10 | unreachable | unreachable | unreachable |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | no | no |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | no |
| F18 | yes | no | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | no | yes |
| F27 | yes | no | no |
| F30 | yes | yes | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | no | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From score JSON intent_recovery.total_possible_weight |
| Reachable gold whys | 47 | S whys always included; F whys included only when feature captured |
| Excluded unreachable feature whys | 2 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 81 / 146 | Sum of weight x recovery-score over included whys |
| Whys with reconstruction evidence | 33 | Exact why evidence present in frozen reconstruction |
| Whys with PLAN grounding | 35 | Exact plan grounding present |
| rule_without_why cases | 12 | Mechanism survived without gold rationale |
| plan_only_not_reconstructed cases | 2 | PLAN preserved rationale/principle but B did not reconstruct it |
| ungrounded_reconstruction cases | 0 | No included why was credited from ungrounded rationale |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 28 | 51.9% |
| Affective whys | 92 | 53 | 57.6% |
| Weight-2 whys | 42 | 21 | 50.0% |
| Weight-3 whys | 104 | 60 | 57.7% |
| System-level whys | 28 | 21 | 75.0% |
| Feature-level whys (reachable) | 118 | 60 | 50.8% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Affective whys recovered 53/92 weighted points (57.6%); functional whys recovered 28/54 (51.9%). The gap is small; both types were limited by missing second-order rationale layers.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 60/104 (57.7%); weight-2 whys recovered 21/42 (50.0%). The planner and reconstructor preserved many primary mechanisms, but downstream consequence layers were frequently compressed away.
- **System-level vs feature-level.** System-level fidelity (75.0%) was much higher than feature-level fidelity (50.8%). The PLAN carried cross-cutting architecture well; feature-specific exceptions like F29, F33, F38, and F39 often survived only as rules.
- **Multi-layer pattern.** Layer 1 often survived, layer 2 sometimes survived when it was technical, and layer 3 was the most commonly lost layer. Examples: F2 lost the too-fast/too-slow failure band, F9 lost the no-toast/canned-cue consequence, and F27 lost the future-erosion foothold.
- **Subdomain pattern.** Server/sync, privacy telemetry, and accessibility narration were the strongest areas. Social and field-notebook edge rationale had more mechanism-only rows: F22, F33, F38, and F39.
- **Evidence-bound effects.** Twelve reachable feature whys were marked rule_without_why. Two system principles, S2 and S4, were plan-only at the system level: the PLAN embodied them cross-cuttingly, but the frozen reconstruction did not name their rationale.

The failure shape suggests a high-capacity planner that captured most implementation surface and a reconstructor that could infer many first-order motives, but the plan did not consistently encode the hidden product-risk arguments that the gold whys test.

## 4. Recommendations for v2 hardening

- Keep the evidence-bound operator. It separated genuine rationale recovery from rule copying in rows like F8, F31, F34, and F39.
- Add more targeted feature-level exception whys. The headroom additions worked: F29-F40 exposed losses that the original broad system whys would have hidden.
- Preserve multi-layer scoring for high-risk whys, but consider asking scorers to separately count lost L1/L2/L3 layers. This run shows downstream consequences are the most fragile layer.
- Clarify capture guidance for documentation-like rows such as glossary and principle callouts. I leaned inclusive on #3 and #7, but those calls are more subjective than API or UI feature rows.
- Keep the system-level cross-cutting bar at three inherited decisions, but require a short appendix as here; it made S2 and S4 transparent despite partial reconstruction.

## 5. Methodology caveats

- **Fresh-context fidelity.** The assignment stated fresh context and the audit found no gold-ID leakage or rubric vocabulary in the reconstruction. I did not spawn subagents or background agents.
- **Single-run-at-temperature limitation.** This is one candidate/reconstructor path; there is no variance estimate from this slot alone.
- **Borderline capture calls.** Inclusive capture affected #3, #7, #20, #31, #55, #58, #83, #84, #94, and #104. The missed features were: #37 No "Welcome back!" toast or banner; #44 Settle is opt-in; #51 Background-tab pause; #52 Click-anywhere-to-undo settle; #65 Top bar auto-fades when cursor idle; #68 Empty-aviary state; #87 Privacy policy link in account settings; #88 Email change flow; #116 Browser support matrix.
- **System-level cross-cutting.** S2 and S4 depended on the three-feature cross-cutting judgment: the PLAN preserved them, but B did not clearly identify their system-level rationale.
- **Confabulation cases.** I found no included why whose reconstruction evidence was ungrounded in the PLAN; ungrounded_reconstruction_count is 0.
- **Evidence-bound denials.** Several plausible v1-style matches were denied because they were mechanism-only: F5, F6, F8, F29, F31-F35, and F38-F39.
- **Operational compromise.** Timing data existed for phase 1 and phase 2A only; I included those bounded fields in the score JSON and did not fabricate phase 2B timings.

End of report.
