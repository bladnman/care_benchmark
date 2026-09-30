# REPORT - CARE run 001
> Variant v06 evidence-bound clean + targeted gold headroom. Every S1-S9 and F1-F40 row includes denominator status, frozen-reconstruction evidence, PLAN grounding, rule-without-why status, and recovery.
---

## 1. Headline
| Score | Value |
|---|---|
| Planning quality | **98.3%** |
| Intent fidelity | **79.6%** |
| Combined quality | **9813** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **79.0%**

**(Planning, fidelity) coordinate:** `(98.3, 79.6)` - plot on a 2D scatter with both axes 0-100.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-30T01:10:31Z |
| Candidate model | gpt-6.1-sol |
| Candidate effort | medium |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **118 / 120** = **98.3%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **118** | **98.3%** |

Per-feature detail:

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in outcome: browser aviary with persistent birds and attention-shaped expressiveness. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Plan makes first meaningful frame an ongoing place and forbids canned/loading theatrics. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Plan forbids banners, unsolicited visit notice, welcome copy, and notification loops. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Plan separates naturalist observation surfaces from matter-of-fact system errors/settings. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Plan explicitly excludes gamification, Tamagotchi harm, public discovery, profiles, chat, rankings, and loops. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Plan starts with two, caps at seven, preserves one no-chrome scene. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | Plan uses domain terms but does not specify a glossary/domain-definition deliverable. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Presence requires visibility, focus, and recent pointer/key activity. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Plan separates persisted traits/filter state from mood and recent-attention envelopes. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Plan defines settle stream quieting, undo/reengage, and equivalent close treatment. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Plan uses five persisted traits and names boldness, warmth, vocal/color/trust/curiosity expressions. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Plan specifies low-pass filtered input and additive nonnegative deltas. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Plan names seven-day and 21-day synthetic calibration targets. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Plan enforces nonnegative deltas and no harm/guilt from absence. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Plan defines five moods and a baseline resampling timer near 24 hours. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Plan uses time of day, reactions, weather, personality, and recent attention. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Plan requires server-planned descriptors and client synthesis, no recordings/loops. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Plan preserves immutable signature anchors and recognizability. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Plan uses independent descriptors with variation and gain headroom. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Plan ties trait expression to call propensity/variation and vocal expression. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Plan lists scan, preen, tilt, shuffle, rest, flight, breathing, and weight shifts. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Plan makes mood generate pose/perch trajectories rather than labels. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Plan includes two starter birds from six species and six silhouettes/signatures. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Plan covers starter adoption/naming and PATCH rename endpoint. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Plan starts with two distinct system-selected species. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Plan enforces cap seven in adoption and verification. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Plan uses age thresholds and rejects attention/reward acceleration. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Plan persists traits/filter state server-side and forbids regeneration. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Plan says login is not a reset trigger and mood must not reset by login. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Plan schedules call replies, alarm nudges, and choruses. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Plan uses stable UUIDs, immutable identity seeds, and no silent replacement. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Plan forbids numeric traits in APIs, DOM, ARIA, logs, debug, exports, and telemetry. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Plan creates owner stream return event and greeting timeline within 1-2 seconds. |
| 34 | Greeting variation by absence length | interactions.md | yes | Plan varies short and long returns differently. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Plan weights greeter by boldness, warmth, and mood. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Plan selects one initial greeter and offsets other responses. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Plan forbids welcome banners, toasts, and generated welcome-back copy. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Plan describes listen-in gain ramps for focused bird. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Plan reduces other birds to audible ambient, never zero. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Plan covers offer reactions including seed/song/pool and server-selected receiver. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Plan ties offers to server reaction, mood behavior, and curiosity nudge. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Plan sets 3-minute per-receiving-bird cooldown across devices/types. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Plan quiets stream and sets session-scoped lighting override. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Plan treats closing without settle as identical for drift and no missing-goodbye notice. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Plan generates naturalist entries from noteworthy canonical facts. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Plan gates routine entries to at least 48 hours, about one per three days. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Plan makes observations immutable with no edit/delete/annotation endpoint. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Plan records qualifying owner attention intervals and uses them for drift. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Plan requires visible + focus + recent pointer/key activity. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Plan excludes streaks, visit-frequency surfaces, session-count entries, and user-frequency prose. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Hidden tabs stop animation/calls; server simulation continues. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Plan gives five-second scene click and keyboard undo/reengage. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Plan keeps horizontal fitted scene with no pan/scroll/crop. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Plan includes three perch zones and depth scale. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Plan forbids user placement and uses server-selected perch/mood behavior. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Plan stores an account IANA timezone and local-time lighting. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured inclusively via local-time lighting, dusk mood bias, and lighting transitions; exact warmer/call-quiet wording is compressed. (borderline inclusive call) |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured inclusively via dusk/drowsy bias and a nightjar-like species retaining night activity. (borderline inclusive call) |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Plan schedules rare short rains and gentle wind. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Plan says rain temporarily reduces call propensity without changing vocal personality. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Plan includes client-only leaves, feathers, parallax, and ornament control. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Plan uses subtle/autonomous parallax and removes it in reduced motion. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Plan keeps controls in a thin top bar and menus/settings outside the scene. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Plan defines four required top-bar icon groups and subordinate menus. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Plan fades top bar after four seconds of pointer inactivity with focus exceptions. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Plan evaluates timelines at first paint; returning bird is mid-preen. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Plan uses soft sky/faint ambient motion, no spinner/fade/static/welcome. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Plan allows true empty scene and gentle fly-in only at initial adoption. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Plan uses muted palettes, calm sky, no UI accent chrome in scene. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Plan uses safe insets, responsive spacing, fitted narrow portrait, and no crop/offscreen overflow. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Plan includes email magic links and excludes passwords/SSO. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Plan sets 15-minute link expiry. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Plan has one canonical aviary per account and excludes shared ownership. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Plan uses UUIDs and confines email ciphertext/digest. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Plan schedules 60-second ticks for every non-deleted aviary. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Plan fetches snapshots on navigation, resume, visibility return, and cadence. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Plan interpolates positions/poses from absolute timeline timestamps. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Plan makes server canonical state and phone/laptop revisions identical. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Plan forbids client absolute values and last-write-wins personality updates. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Plan makes only the simulation role/tick write personality. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Plan uses matter-of-fact inline system messages and retry actions. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Plan includes device session records and GET/DELETE session revocation. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Plan implements export job, signed link, and defined snapshot contents. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Plan implements soft deletion, recovery, hard purge, and key destruction. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Plan forbids simulation events in telemetry/analytics and any ML-like aggregation. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Plan allowlists aggregate operational metrics and excludes bird/account interaction data. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Plan includes privacy text/settings disclosures but not a privacy-policy link requirement. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Plan verifies pending encrypted address before switching. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Plan creates explicit one-email invitations and no fan-out/public index. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Plan permits visits only through deliberate invitations, with no global public/default surface. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Plan makes visits read-only and excludes visitor events/presence. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Plan excludes owner controls from visitor semantic/route surfaces and rejects visitor capabilities. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Plan excludes profiles, chat, comments, public social surfaces, and visitor active controls. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Plan uses no push/email/badge/toast and a settings-only summary/log. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Plan checks revocation on every request and supports DELETE invitation. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Plan has visit session/log with host account settings visibility. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Plan uses the same renderer/projection/read endpoint for visitors. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Plan excludes public discovery, rankings, public account-state cache, and underlying analytics. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Plan generates lowercase present-tense prose from canonical state. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Plan sets 45-second default, configurable 30-60, one polite live region. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Plan uses naturalist prose and rejects welcome/achievement/session-count language. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Plan replaces motion/parallax/flight with 2-4s cross-fades. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Plan keeps audio, captions, moods, notebook, and drift functional. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Plan includes narration/caption settings and caption prose from actual descriptors. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Plan audits copy/focus/caption states for WCAG AA. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Plan specifies top bar/bird navigation, Enter/Escape, menus, and keyboard tests. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Plan requires visible focus outlines against day/night palettes. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Plan gates initial JavaScript <2MB gzip and targets <250KB critical. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Plan gates p95 <500ms in a fixture. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Plan has 60fps/16.7ms frame budget and soak tests. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Plan defines 30-minute heap/native resource trend tests. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Plan synthesizes descriptors client-side with WebAudio nodes. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Plan forces captions and silence when WebAudio unavailable, no recordings. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Plan includes synthetic browsers and RUM histograms without stable account/device/bird dimensions. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Plan alerts on tick compute p99 over 5s and lag. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Plan supports last two major versions and explains unsupported browsers. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Plan excludes native clients. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Plan excludes achievements, streaks, scores, badges, XP, ranks, tiers, and counters. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Plan excludes hunger, death, distress, meters, and negative absence effects. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Plan excludes public discovery, profiles, chat, comments, rankings, social cursors, and public surfaces. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md §System-level intent: "An ongoing place, not an app starting"; "Procedural recognizability instead of recordings or canned loops." | PLAN.md §1: "first meaningful frame shows an ongoing place"; §8: "already mid-preen"; §9: "No recorded files or loop playback." | yes | yes | no | full | The reconstruction connects continuing place, motion, greetings, procedural calls, and loading behavior. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md §System-level intent: "Quiet product voice and no engagement machinery"; "no textual welcome"; "no unsolicited notice." | PLAN.md §1: "A return is noticed by a bird"; §2: "No unsolicited notice reaches the host"; §10: "Never auto-generate 'welcome back'." | yes | yes | no | partial | Operational refusal of announcements survived; the processed-vs-seen affective layer was thinner. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md §Scene: "Notebook specificity, sparsity, deduplication"; entries from "real canonical state" and "not generic happiness messages." | PLAN.md §8: "specific entries from noteworthy canonical facts"; §10: "lowercase present-tense prose naming birds." | yes | yes | no | full | Specific naturalist observation, named birds, and anti-generic wording were recovered. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md §1: "two system-selected starter birds" and "up to seven"; §8: "do not pan, scroll or crop"; §8: "No hover labels or scene icons." | no | yes | no | partial | The plan preserves the principle across caps, scene, palette, and chrome, but reconstruction did not name it systemically. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md §System-level intent: "matter-of-fact voice" and "naturalist voice for observation surfaces"; §Accessibility: "System errors directly explain action and recovery." | PLAN.md §5: "matter-of-fact voice, not toasts over the aviary"; §10: "observation surfaces use the naturalist voice." | yes | yes | no | full | The voice split and system exception both survived. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md §System-level intent: "Slow, nonpunitive attention"; §Presence: "strict attention calibration with visibility, focus, and recent activity." | PLAN.md §1: "Presence requires visibility AND window focus AND recent pointermove/keypress"; §7: "do not silently broaden eligibility to a tab-open check." | yes | yes | no | partial | Strict presence survived; settle/tab-close equivalence and relationship-shape consequence were less explicit at system level. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md §System-level intent: "Server-authored canonical life"; browser "cannot decide canonical" state; phone/laptop views match. | PLAN.md §3: browser "cannot decide canonical" state; §6: ticks run for aviaries "including those with no clients"; §7: phone/laptop comparison identical. | yes | yes | no | full | Server-side tick, multi-device canonical state, and no-client/client-failure rationale all survived. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md §System-level intent: "Privacy and lifecycle trust over convenience"; interaction logs are "simulation inputs, not a warehouse." | PLAN.md §11: telemetry collector has "no credentials/network access to simulation tables"; "Interaction logs are simulation inputs, not a warehouse." | yes | yes | no | full | Private relationship data and technical telemetry separation were recovered. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md §System-level intent: "Accessibility is a first-class deliverable"; experiences "ship together" and should feel like observing birds. | PLAN.md §1: "visual, audio, narration, caption, and reduced-motion experiences ship together"; §13: "Do not postpone accessible architecture"; §10: "feels like observing birds." | yes | yes | no | full | Charm, equal-quality modes, and ship-with-v1 timing were all present. |

For multi-layer system-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | yes |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: Outcome invariant, mid-action bootstrap/loading, procedural calls, greeting variation, narration/reduced-motion/captions.
- S2: No welcome banner/toast, settings-only visit summary, no streaks/session counts, field notebook avoids user-frequency prose.
- S3: Naturalist notebook/narration, named birds and observed names, hidden numeric traits, no public comparison.
- S4: Two starters/seven cap, one fitted horizontal scene, top-bar/no scene chrome, compact renderer/bundle discipline.
- S5: Naturalist observation surfaces, matter-of-fact errors/settings, direct auth/sync errors, prohibited welcome/achievement copy.
- S6: Strict presence gate, drift input/caps, settle/close equivalence, no streaks/Tamagotchi/guilt surfaces.
- S7: Server-side tick, server-only personality writes, client snapshot rendering, multi-device canonical state, no LWW.
- S8: Telemetry table separation, no simulation events in analytics, export/deletion boundaries, invitee PII scoping.
- S9: Narration, reduced motion, captions, keyboard/focus, manual assistive testing, ship-together release sequence.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **79.0%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md §Presence: "strict attention calibration with visibility, focus, and recent activity"; no "tab-open duration." | PLAN.md §1: "Presence requires visibility AND window focus AND recent pointermove/keypress"; §7 forbids tab-open broadening. | no | partial | Recovered precise gate and tab-open failure; downstream silent drift-corruption layer was not fully reconstructed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md §Simulation: "Drift function with filtered input, additive nonnegative deltas, caps, saturation, and synthetic calibration." | PLAN.md §6: "low-pass filter"; seven-day and 21-day calibration targets; no click-count reward. | no | partial | Low-pass slowness survived; instruments-vs-user gap and Tamagotchi/screensaver endpoints were mostly plan-only. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md §System-level intent: absence "cannot lower a trait, harm a bird or produce a guilt surface"; §Simulation: "no negative drift." | PLAN.md §1: "Absence cannot lower a trait, harm a bird or produce a guilt surface"; §6: "never decreases them." | no | partial | Nonpunitive monotonic drift survived; the explicit symmetric-drift/Tamagotchi implementation contrast was thinner. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md §System-level intent: "Procedural recognizability instead of recordings or canned loops"; §Audio: no "recorded files or loop playback." | PLAN.md §1: "Calls are always procedural"; §9: "No recorded files or loop playback" and independent chorus descriptors. | no | full | Recovered no-loop aliveness, chorus variation, and WebAudio/fallback implications. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md §Outcome: mood changes "generate gentle pose/perch trajectories, not numerical labels." | PLAN.md §6: "Mood changes generate gentle pose/perch trajectories, not numerical labels." | no | full | The visible-mood-without-labels rationale survived. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md §Audio: chorus variation keeps "seven signatures distinguishable"; §Risks: recognizability panel at each count. | PLAN.md §9: "Cap simultaneous voices" so "seven signatures remain distinguishable"; §15: recognizability panel at each count. | no | full | The cap's recognizability rationale survived, though not with the empirical-ceiling wording. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md §System-level intent: UUIDs/personality/call identity "survive renames, deployments and migrations"; recovery resumes "same birds and vectors." | PLAN.md §1: persisted personality survives migrations; §4: bird has stable UUID, traits, filter state; §11 recovery restores same IDs/vectors. | no | full | Recovered canonical persisted vector, relationship continuity, and downstream sync/no-LWW protection. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md §System-level intent: "Hidden inner state stays hidden"; no numbers in APIs, DOM, ARIA, logs, exports, telemetry. | PLAN.md §1: hidden personality numbers never appear in user-visible API, DOM, ARIA, logs, debug, exports; §4 no stat interface. | no | full | Recovered the anti-stat-interface purpose. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md §Simulation: greeting is a bird noticing return within "1-2 seconds" with weighted variation; §Review: short/long absences and moods are varied. | PLAN.md §6: one initial greeter by boldness/warmth/mood; shorter/longer absences differ; no simultaneous cue. | no | partial | Initial greeter and variation survived; the generic-arrival-animation downstream warning was not fully reconstructed. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md §System-level intent: system "never welcomes the user with a banner"; §Accessibility: avoid "welcome back." | PLAN.md §1: system never welcomes with banner; §10: never auto-generate "welcome back"; §8 no welcome/loading theatrics. | no | partial | The rule and variants survived; the 'system told me I was back' reframing layer was under-articulated. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md §Simulation: settle has "identical drift treatment to close/lease expiry without missing-goodbye copy." | PLAN.md §6: "Closing without settle closes/lets the same lease expire, with identical drift treatment and no missing-goodbye notice." | no | full | The no-penalty/ceremony rationale was recovered through drift equivalence. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md §Outcome: notebook uses "noteworthy canonical facts, not session timestamps or visit streaks"; §Scene: not "generic happiness messages." | PLAN.md §8: entries from noteworthy canonical facts; at least 48 hours between routine entries; no edit/delete/annotation endpoint. | no | partial | Naturalist specificity and rarity/read-only survived; the notebook-as-concentrated-voice layer was thinner. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md §Presence: strict gate and leases prevent "crashed or sleeping clients from accruing hours"; no offline presence replay. | PLAN.md §7: all three conditions; server bounds duration; lease expires after 30 seconds; no backfill/offline replay. | no | partial | Implementation precision survived; silent population-wide drift corruption was not fully recovered. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md §System-level intent: excludes "streaks, visit-frequency surfaces"; §Scene: no "session timestamps or visit streaks." | PLAN.md §1 excludes streaks and visit-frequency surfaces; §8 avoids session timestamps/visit streaks; §10 avoids session-count entries. | no | partial | The named refusals and adjacent disguises survived; managing-a-number rationale was weaker. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md §Scene: timeline evaluation means bird is "already mid-preen"; loading has "no spinner, fade-from-static, welcome, or invented state." | PLAN.md §8: returning bird already mid-preen; loading uses soft sky/faint motion, no spinner/fade/static/welcome; §6 server timelines. | no | full | Recovered continuing-place frame, server-time implementation, and quiet-field/no-spinner rationale. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md §Data: email ciphertext confined to Account; digest restricted so email is not "a service identifier or log key." | PLAN.md §4: email ciphertext lives once; digest is restricted to auth lookup; §3 job payloads use synthetic IDs, never email. | no | partial | Synthetic-ID and PII-leakage layers survived; retrofit/compliance consequence was not reconstructed. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md §Simulation: 60-second ticks for aviaries "including no-client aviaries"; "ongoing server-side place." | PLAN.md §6: schedule every 60 seconds including no clients; §3 browser cannot decide canonical state; §7 phone/laptop identical. | no | full | Recovered server tick, multi-device coherence, and client-simulation failure avoidance. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md §HTTP: no trait, mood, or arbitrary patch accepted; server receipt order supplies state order; only simulation role writes personality. | PLAN.md §1: no client absolute values or last-write-wins personality updates; §5: clients cannot patch traits; §6 additive server-authored delta. | no | partial | Implementation rule survived; the invisible cross-device overwrite example was not fully recovered. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md §HTTP: "Matter-of-fact inline system messages and retry actions" avoid "toasts over the aviary"; §Voice: errors directly explain action/recovery. | PLAN.md §5: Display persistent inline system messages and retry actions in matter-of-fact voice, not toasts; §10 system errors explain action/recovery. | no | full | Recovered the system-clarity exception to naturalist charm. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md §Privacy: telemetry collector lacks simulation-table access; interaction logs are "simulation inputs, not a warehouse"; synthetic-only dashboards. | PLAN.md §11: collector has no access to simulation tables; never log bird/name/traits/presence; all drift dashboards synthetic-only. | no | full | Recovered simulation-only use, private-relationship stance, and data-pipeline enforcement. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md §System-level intent: visits are "explicitly invited read-only visits"; visitor routes cannot create presence or events. | PLAN.md §5: visit snapshot is same projection, excludes notebook/private settings; visitor heartbeat only updates approximate log duration; §14 visitor routes cannot create presence/events. | no | full | Recovered observation-not-co-presence and no visitor drift. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md §Decisions: settings-only visit summary has "no badge, toast, push, email, or unsolicited host notice." | PLAN.md §2: no badge, toast, push or email; §5 settings-only visit summary; no unsolicited notice reaches host. | no | full | Recovered quiet transparency rather than an attention loop. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md §Outcome: avoids leaderboards, recommendation features, engagement loops, and cross-account analytics. | PLAN.md §1: excludes public discovery, rankings, and cross-account analytics; §15 future engagement features blocked by metric allowlists. | no | full | Recovered the refusal of public comparison and underlying aggregation. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md §Accessibility: narration is lowercase present-tense prose from the same snapshot; review asks whether it feels like observing birds. | PLAN.md §10: narration from same snapshot/events, lowercase present-tense prose; review whether it feels like observing birds. | no | full | Recovered prose, equal affective quality, and anti-ARIA/state-list implementation rule. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md §Scene: reduced motion keeps audio, captions, moods, notebook, and drift functional; §System-level: accessibility modes remain alive. | PLAN.md §8: cross-fades replace motion; audio/captions/moods/notebook/drift remain functional; §1 modes ship together. | no | full | Recovered different-rendering-not-stripped-fallback across all layers. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md §Performance: first bird measured at actual renderer draw; 500ms target via bootstrap, compact snapshot, and minimum shapes. | PLAN.md §12: first bird measured at actual renderer draw; p95 <500ms; compact snapshot and minimum species shapes. | no | full | Recovered the affective performance threshold and implementation implications. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md §Outcome: excludes rankings, badges, streaks, notification loops, hunger, death, distress, and meters; avoids engagement loops. | PLAN.md §1 excludes meters, achievements, streaks, badges; §10 avoids achievement language; §15 blocks future engagement features. | no | partial | The loud non-goal and future erosion survived; cheap/tempting adjacent-product layer was less explicit. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md §System-level intent: absence "cannot lower a trait, harm a bird or produce a guilt surface"; no negative drift. | PLAN.md §1 excludes hunger, death, distress, meters; absence cannot harm; §6 never encodes absence as distress. | no | full | Recovered observational, non-custodial, nonpunitive relationship. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | RECONSTRUCTION explicitly says "Two system-selected starter birds from six species: NOT RECOVERABLE FROM PLAN"; rule captured, why not recovered. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md §Outcome: time-only eligibility persists, declining costs nothing, and expansion occurs "without teaching attention-for-rewards." | PLAN.md §2: thresholds by aviary age, not countdown/reward/scene badge; "without teaching attention-for-rewards"; no frequent-attendance acceleration. | no | full | Recovered age-as-time, rejected reward loops, and economy-erosion rationale. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md §System-level intent: "Durable private continuity"; UUIDs/personality/call identity survive migrations; no rollback silently replaces birds. | PLAN.md §1 bird UUIDs/personality/call identity survive; §4 stable UUID/identity seed; §13 no adopted bird IDs reset. | no | full | Recovered same-individual continuity, data identity, and retroactive relationship protection. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md §Simulation: mood transitions are not "login resets"; tests require "mood not reset by login." | PLAN.md §6: Login is not a reset trigger; §14 mood not reset by login. | no | full | Recovered no-neutral-reset continuity. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md §Outcome: notebook uses noteworthy canonical facts and entries are immutable; §Scene: no edit/delete/annotation surface. | PLAN.md §4: entries immutable; §8: no edit/delete/annotation endpoint and observations from canonical facts. | no | full | Recovered observer-record immutability rather than user curation. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md §Outcome: export gives user access to bird identities, names, species, mood, notebook, and settings while honoring hidden vectors. | PLAN.md §2: export identities, names, species, mood, notebook, settings; §5 consistent revision and signed link. | no | full | Recovered quiet user access to the aviary copy, albeit with emphasis on hidden-vector conflict. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md §Outcome: lifecycle trust; recovery restores same IDs/vectors; hard deletion purges state and destroys account key. | PLAN.md §11: 30-day recovery restores same IDs/vectors; deadline purges relational state; hard deletion destroys per-account key. | no | full | Recovered regret window, privacy hard-delete, and all-records-together deletion. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md §Privacy: operational telemetry allowlist and separate collector without simulation-table access; no bird/name/trait/account history logging. | PLAN.md §11: explicit metric allowlist; collector has no simulation credentials; never logs bird IDs/names/traits/presence/account histories. | no | full | Recovered technical telemetry boundary. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md §System-level intent: "Explicit, bounded social access"; no public discovery; §HTTP: invite one email, no fan-out. | PLAN.md §5: explicitly invite one email; no public index/fan-out; §4 invitee email scoped only to Invitation. | no | full | Recovered deliberate named sharing and no discoverable ambient exposure. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md §Privacy: visit logs provide "host transparency"; §Decisions: summary/log is on demand with no badge/push/email. | PLAN.md §2: settings-only visit summary, no badge/toast/push/email; §11 visit logs retain invitation identity, approximate duration, timestamps. | no | full | Recovered transparency without notification loop. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Reconstruction preserves same renderer/projection as a rule but not the witness-real-birds/no-show-off rationale. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md §Accessibility: default cadence 45 seconds, configurable 30-60; one polite live region with latest-state queue, avoiding overload. | PLAN.md §10: Default idle cadence 45 seconds, configurable 30-60; one polite live region; coalesces events rather than overloading. | no | full | Recovered slow rhythm, queue overload, and sparse observational prioritization. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | no | no |
| F3 | yes | no | yes |
| F4 | yes | yes | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | no |
| F10 | yes | no | yes |
| F12 | yes | no | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | no | yes |
| F30 | yes | yes | yes |
| F31 | yes | yes | yes |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON `gold_why_totals` |
| Possible total weight | 152 | From score JSON `intent_recovery.total_possible_weight` |
| Reachable gold whys | 49 | S whys always included; all 40 F anchors reachable |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 121.0 / 152.0 | Sum of weight x recovery-score over included whys |
| Whys with reconstruction evidence | 46 | Exact rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 47 | Exact plan grounding present |
| `rule_without_why` cases | 2 | F29 and F39 |
| `plan_only_not_reconstructed` cases | 14 | Partial rows where plan carried fuller rationale than reconstruction |
| `ungrounded_reconstruction` cases | 0 | No major ungrounded rationale assertions found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 42.0 | 77.8% |
| Affective whys | 98.0 | 79.0 | 80.6% |
| Weight-2 whys | 44.0 | 39.0 | 88.6% |
| Weight-3 whys | 108.0 | 82.0 | 75.9% |
| System-level whys | 28.0 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 124.0 | 98.0 | 79.0% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional recovery was 42.0/54.0 (77.8%); affective recovery was 79.0/98.0 (80.6%). Functional misses clustered in calibration/concurrency details (F1, F2, F13, F16, F18), while affective misses clustered in deeper exception stories (S2, S4, F10, F12, F29, F39).
- **Weight-3 vs weight-2.** Weight-2 whys recovered 39.0/44.0 (88.6%), while weight-3 whys recovered 82.0/108.0 (75.9%). The high-weight rows exposed layer loss: primary implementation usually survived, downstream consequence often did not.
- **System-level vs feature-level.** System-level recovery was slightly higher (82.1%) than feature-level recovery (79.0%). The plan encoded cross-cutting philosophy very well, but reconstruction sometimes compressed feature exceptions into rules.
- **Multi-layer recovery patterns.** Layer 1 was most durable. Layer 2 and Layer 3 dropped in rows such as F2, F10, F13, F16, and F18, where the plan had specific failure-mode detail but reconstruction summarized the mechanism.
- **Subdomain patterns.** Audio, accessibility, privacy, lifecycle, and server-side simulation scored strongly. Social sharing and onboarding had the clearest rule-without-why losses: F29 and F39.
- **Evidence-bound effects.** v06 denied credit where plan coverage was high but frozen reconstruction did not carry rationale evidence. F29 and F39 are the cleanest examples; S4 also illustrates plan-only system preservation.

What the failure shape suggests: this candidate writes extremely actionable implementation plans, but some affective intent remains embedded as constraints rather than as durable rationale.

---

## 4. Recommendations for v2 hardening

- Keep targeted headroom whys like F29-F40. They uncovered remaining compression even in a high-planning-quality run.
- Add or retain downstream-consequence layers for multi-layer whys. This run shows that downstream failure modes are the first rationale to vanish.
- Keep evidence-bound scoring. Without it, strong plan coverage would obscure rule-without-why losses.
- Consider adding a few more social/onboarding exception whys, because those are easy to implement mechanically while losing the product relationship rationale.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction shows no gold IDs, no rubric vocabulary, and follows the PLAN section order. The validity audit verdict is PASS.
- **Single-run-at-temperature limitation.** This is one run, so there is no variance estimate.
- **Borderline capture calls.** Features 57, 58, 90, and 41 were read inclusively. Only 57 and 58 are marked as borderline in the feature table; none affect feature-why reachability.
- **System-level cross-cutting.** S4 is the notable subjective call: the plan preserved restraint, but the reconstruction did not identify it as a system-level principle, so it scored partial.
- **Confabulation cases.** No major ungrounded reconstruction cases were found; ungrounded reconstruction count is 0.
- **Evidence-bound denials.** F29 and F39 were captured as rules but scored none for why recovery because the reconstruction did not recover the rationale.
- **Rule-without-why cases.** Two included feature whys were rule-without-why: starter birds not catalog choices, and visitor sees actual aviary/no show-off mode.

End of report.
