# REPORT - CARE run 001

> Phase 2B scoring report for wave_003 slot 001. Companion artifacts: RECONSTRUCTION.md, run_001.json, and REPORT.html.
>
> Variant v06 evidence-bound clean + targeted gold headroom. Every S1-S9 and F1-F40 row separates denominator status, reconstruction evidence, plan grounding, rule-without-why, and recovery.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **98.3%** |
| Intent fidelity | **83.6%** |
| Combined quality | **9817** |

**Diagnostic split:**

- System-level fidelity: **89.3%**
- Feature-level fidelity: **82.3%**

**(Planning, fidelity) coordinate:** (98.3, 83.6).

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T02:43:53Z |
| Candidate model | gpt-5.5 |
| Candidate effort | extra-high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what did not

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

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured by product framing: browser-only single-user virtual aviary with procedural scene and no gamification. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured as affective constraints treated as hard technical requirements and first-frame already-in-motion goal. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured through no welcome surfaces, no badges, no pushed visit alerts, and bird greeting as welcome. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by naturalist product voice and matter-of-fact system/error/settings surfaces. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in explicit out-of-scope game, Tamagotchi, notification, and public social exclusions. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by two starter birds, hard cap seven, one screen, and calm/no-chrome scene constraints. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | Not captured as a glossary/documentation feature; terms are used and modeled but no glossary deliverable is planned. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured by precise visible/focused/recent-activity presence and slow drift input. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by hidden slow personality vectors and fast-timescale mood state. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by settle gesture as user-initiated session end/evening lighting with undo. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured with all five hidden server-side vector fields. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by slow low-pass filter over host presence and interactions. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by explicit one-week instrument and three-week felt calibration. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by no downward movement from neglect and ambient quietness after absence. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by fast-timescale mood model with daily/time/weather inputs and persistence. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by mood transitions from recent host events, time, weather, personality, and bird influence. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by WebAudio motif libraries and runtime grammar descriptors. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by call_signature_seed and recognizability tests up to seven birds. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by chorus relationships, mix buses, and recognizably non-looped audio tests. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by vocal-frequency trait modulating timing, calls, and chorus participation. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by normal motion system: preen, scan, head tilt, body shuffle, call posture. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by mood visible through motion, perch, calls, and offer reactions. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by small coherent species pool of about six species. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by user-assigned, renameable bird names and rename endpoint. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured by two server-selected starter birds with naming, not a catalog. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by hard cap of seven birds. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by age-based bird additions, not visit count, score, payments, or engagement. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by persisted server-side personality vectors and server-only writes. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by mood persists across sessions and no neutral reset on open. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by bird-to-bird influence, chorus, and reactions. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by stable bird_id preserved across rename, sync, migrations, and asset updates. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by no personality numbers in UI/debug/stats, except private export file. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by one bird greeting within first second or two, varied by absence, mood, and personality. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by absence duration buckets shaping greeting form. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by boldness/social warmth weighting in greeter selection. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured by one primary greeter and staggered secondary reactions. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by no textual welcome, no toast/banner/modal, and no text accompanying greeting. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by click/keyboard focus and focused bird gain ramp. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured by non-focused birds ramp down but never silent. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by offer types seed, song fragment, and still pool. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by reaction candidates based on mood, curiosity, proximity, and state. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by per-bird/per-offer cooldowns of a few minutes. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by settle gesture with slow evening lighting and quiet calls. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured by closing tab without settle being equally valid. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by sparse naturalist observations and template/lint guardrails. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by sparse cadence, few-days interval, and meaningful-event filters. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by no edit/delete endpoints and read-only forever. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured by presence pings/windows as dominant drift input. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by visible + focused + recent pointer/key activity rule. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by no streaks, visit calendars, visit counts, or progress surfaces. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured by hidden tabs stop rendering/presence; server canonical state continues. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured by five-second undo via any click in aviary. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by responsive single-scene aviary with no panning, scrolling, or zooming. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by three depth-aware perch zones and front/middle/back fields. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by perch as simulation output and no bird placement endpoints. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured by local timezone day/night and mood inputs. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured by settle/evening lighting, day/night palette, and quieter calls. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by night variants and night activity profile; less specific but adequate. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by rare quiet ambient weather and weather state. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by weather mood nudges and weather affecting mood/call behavior. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by ambient leaves/feathers drift client-side. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured by depth-aware scene and subtle layout; no panning/zooming keeps it restrained. |
| 63 | No UI chrome inside aviary view (icons live in thin top bar) | aviary_layout.md | yes | Captured by top bar outside scene and no controls/labels inside scene. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by top bar contents exactly. |
| 65 | Top bar auto-fades when cursor idle | aviary_layout.md | yes | Captured by fade nearly transparent after cursor stillness and return on activity. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by first frame birds mid-action and no entry sequence. |
| 67 | Loading state is quiet field, not spinner | aviary_layout.md | yes | Captured by quiet-field fallback and no spinner. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured by quiet field between adoption and first bird arrival. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured by calm naturalist palette and no saturated UI accent colors. |
| 70 | Aviary scene responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by layout solver and viewport tests preventing cropped birds. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by email magic-link auth and out-of-scope passwords. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by 15-minute expiration. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by single-user accounts and one aviary per account. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured by synthetic UUID and no email-derived identifiers. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured by approximately once-per-minute server-side tick. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by snapshot refresh on visibility return. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by client interpolation between snapshots. |
| 78 | Multi-device sync (state canonical server-side) | accounts_sync.md | yes | Captured by all devices reading same canonical server state. |
| 79 | Last-write-wins forbidden for personality state | accounts_sync.md | yes | Captured by no client absolute personality writes and append-only events. |
| 80 | Conflict resolution: server tick only writer of personality drift | accounts_sync.md | yes | Captured by simulation worker only writer and tick consuming event log. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by matter-of-fact conflict/session/snapshot errors. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by revocable per-device sessions. |
| 83 | Account export (download JSON snapshot of aviary) | accounts_sync.md | yes | Captured by account export JSON snapshot and export job. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured by 30-day soft deletion and hard deletion. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by per-bird data not used for ML/training/analytics. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured by aggregate operational telemetry only. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Not captured as an account-settings link; plan tracks accepted privacy version but does not specify a visible policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by pending email fields and email-change endpoint. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured by email-based revocable invitations. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by invites default nonexistent/off by default. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured by read-only visitor snapshot and no simulation events. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by visitor cannot send offers, listen-in, settle, presence, or notebook-affecting events. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by exclusions of chat, comments, avatars, profiles, and visitor cursors. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured by no pushed visit notification and default-off optional toggle. |
| 95 | Visit revocation (host can revoke invite any time) | social_optional.md | yes | Captured by immediate revocation endpoints. |
| 96 | Visit log (host can see who visited and when, in settings) | social_optional.md | yes | Captured by host visit log in settings. |
| 97 | Visitor sees host aviary as it is (no show-off mode) | social_optional.md | yes | Captured operationally by read-only host aviary snapshots, though show-off rationale is thin. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured by no leaderboards/discovery/public aviaries. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured by naturalist prose narration from snapshot state. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured by 30-60 second idle cadence and queue control. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by naturalist present-tense narration, no raw labels/event logs. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured by cross-fades between still poses and perches. |
| 103 | Reduced-motion mode preserves charm (not stripped fallback) | accessibility_perf.md | yes | Captured by designed alternate register preserving calls, mood, identity, captions, narration. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured by optional captions generated from call grammar and default fallback. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by WCAG AA text requirements. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by complete keyboard flow. |
| 107 | Focus indicators visible against aviary states | accessibility_perf.md | yes | Captured by focus indicators tested against bright/dim/rain/night. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by hard V1 budget under 2MB gzipped. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured by first-bird under 500ms budget. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by 60fps idle target. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by no memory growth and 30-minute soak tests. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by WebAudio procedural synthesis and no recorded assets. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by graceful silence and captions when WebAudio unavailable/blocked. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by aggregate RUM and synthetic monitoring. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by tick p99 alarm at five seconds. |
| 116 | Browser support matrix (last 2 majors Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured by current major browser optimization and beta browser-support matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by browser-only and native app exclusion. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured by absolute gamification exclusions. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured by Tamagotchi exclusions and no punishment for absence. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by public social/network exclusions. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **89.3%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level: "first visible frame feel already alive"; "small living window that continues" | PLAN.md §§1,8,12: "already in motion"; "No spinner"; procedural calls and quiet-field first paint | yes | yes | no | full | All three aliveness layers survive across rendering, audio, loading, and server continuity. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level: "notices them when they arrive"; visits/logs are "never pushed" | PLAN.md §§2,6,8: no welcome surfaces, no badges, no pushed visit alerts, bird greeting only | yes | yes | no | partial | Operational refusal of announcements survives; the processed-vs-seen affective layer is thinner. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md: notebook is "sparse, stable naturalist observation" and narration/captions avoid raw labels | PLAN.md §§6,10,17: naturalist notebook/narration/caption templates and no generic event-log phrasing | yes | yes | no | full | Specific naturalist prose, named birds, and anti-generic surfaces are preserved. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md §§2,8,17: two starter birds, max seven, single scene, no panning, no scene customization, thin top bar | no | yes | yes | partial | The plan preserves restraint cross-cuttingly, but the reconstruction does not name the depth-over-variety why. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level: "Separate naturalist product voice from system voice" | PLAN.md §§1,5,7,10: naturalist product surfaces; matter-of-fact auth, errors, settings, export, deletion | yes | yes | no | full | Voice split and error-context exception are explicitly preserved. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System-level: "presence as a precise, quiet signal rather than engagement" | PLAN.md §§1,4,6,14: visible/focus/activity conjunction, slow drift input, settle/tab-close equivalence, background-tab rejection | yes | yes | no | full | Presence precision, anti-inflation, and non-punitive end-of-presence semantics survive. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level: "Keep canonical life on the server" | PLAN.md §§3,6,7: server tick, client snapshots only, server-only personality writes, no last-write-wins | yes | yes | no | full | Server canonicality, sync coherence, and client-state corruption risks are captured. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level: "Enforce privacy by boundaries, not policy text" | PLAN.md §§4,11,12: event log for simulation only; aggregate telemetry; no per-bird analytics/training | yes | yes | no | full | The data-pipeline boundary, not just policy, survives. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level: "Make accessibility the product experience" | PLAN.md §§1,10,13: narration, reduced motion, captions, keyboard, contrast all ship in V1 and preserve charm | yes | yes | no | full | Charm parity, costlier designed accessible surfaces, and V1 shipment all survive. |

Multi-layer system whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | Y | Y | Y |
| S2 | Y | N | Y |
| S6 | Y | Y | Y |
| S7 | Y | Y | Y |
| S9 | Y | Y | Y |

**Cross-cutting evidence appendix:**

- S1: PLAN §§1, 6, 8, 9, 10, 12 tie aliveness to server continuity, first frame, procedural calls, mood motion, quiet loading, reduced motion, and performance.
- S2: PLAN §§2, 6, 8, 13, 17 refuse welcome text, streaks, badges, pushed visits, notification pressure, and gamification flags.
- S3: PLAN §§4, 6, 8, 10, 17 preserve naturalist specificity in notebook, narration, captions, naming, and anti-log lint rules.
- S4: PLAN §§2, 8, 9, 16, 17 preserve two starters, max seven, single scene, thin chrome, no customization, and recognizability tests.
- S5: PLAN §§1, 5, 7, 10, 11 apply naturalist voice to product surfaces and matter-of-fact voice to auth, errors, settings, export, deletion, and accessibility settings.
- S6: PLAN §§1, 4, 5, 6, 14 define presence precisely, use it for drift, reject background tabs, make settle optional, and refuse streak-style progress.
- S7: PLAN §§3, 4, 6, 7, 14 make the server tick canonical, forbid client personality writes, process append-only events, and test multi-device consistency.
- S8: PLAN §§4, 11, 12, 14 enforce synthetic IDs, simulation-only event logs, aggregate-only telemetry, log scrubbing, export, and deletion.
- S9: PLAN §§1, 8, 10, 13, 14 require narration, reduced motion, captions, keyboard navigation, contrast, and assistive-tech testing before launch.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **82.3%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: "Precise presence accounting" rejects background tabs, sleep gaps, stale pings | PLAN.md §§4,5,6: visible + focused + recent activity, calibrated drift, background-tab test | no | full | All three precision/false-presence/silent-corruption layers recover. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: "one-week measurable and three-week felt movement" | PLAN.md §6: low-pass filter, one-week instruments, three-week felt drift, Tamagotchi/screensaver risk | no | full | Calibration and failure-mode band are preserved. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: "no downward movement from neglect" and no "guilt" or "distress" | PLAN.md §§6,16: no negative personality drift; two-week absence stays ambient, not distressed | no | full | Non-punitive drift, Tamagotchi refusal, and absence return behavior recover. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: "avoid recorded loops" and keep calls "varied, recognizable per bird" | PLAN.md §§9,14: WebAudio grammar, no recorded call assets, varied recognizable calls, captions from grammar | no | partial | Runtime procedural rule and WebAudio cascade survive; chorus phase-cancel rationale is not recovered. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md: mood is visible through "motion, perch, calls, and offer reactions" without labels | PLAN.md §§6,8: normal motion keyed by mood/personality and mood output as descriptors, not labels | no | full | Motion-as-mood surface is recovered. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | The cap is captured, but reconstruction says the specific rationale for seven is not recoverable. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md: "Hidden server-side personality vectors" and client cannot write trait values | PLAN.md §§3,4,7: vectors persisted server-side, tick writer only, migrations preserve bird state | no | partial | Canonical persistence and sync consequences recover; losing-vector-as-deleting-bird affect is thin. |
| F8 | personality-vector-never-numerical | 2 | yes | included | RECONSTRUCTION.md: hidden vectors avoid "optimization surfaces" | PLAN.md §§2,4,17: no visible personality numbers except private export file; no optimization surface | no | full | The stat-management danger is recovered in optimization-surface terms. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md: "one primary greeter" varying by absence, mood, personality, history; "No text" | PLAN.md §§5,6: one bird within first second or two; absence/mood/personality variation; no text | no | partial | Selection and variation recover; generic-arrival-animation failure mode is not explicit. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md: return greeting is "recognition without announcement" and "No text accompanies this greeting" | PLAN.md §§2,6,8: no welcome banners/toasts/text; no top-bar notification pressure | no | partial | The no-text welcome surface survives; the deeper reframing/different-product rationale is mostly absent. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md: settle is "not a reward or obligation" and tab close is "equally valid" | PLAN.md §§6,8: closing tab without settle is equally valid; no implication user must settle | no | full | Optional ritual versus chore/punishment recovers. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md: notebook is "sparse, stable naturalist observation" not a "log or achievement feed" | PLAN.md §§4,6,8: lowercase present-tense specific prose, sparse cadence, read-only, no event-log/user-frequency phrasing | no | full | Naturalist prose, anti-log voice, rarity, and read-only status recover. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: validated visible/focused/recent activity and rejects background tabs/laptop sleep | PLAN.md §§4,5,6,14: all three signals, background simulations, silent drift inflation tests | no | full | Implementation precision and false-positive cases recover. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: presence is never "visit counts, streaks, calendars, or user-facing progress" | PLAN.md §§2,4,6,12: no streaks/visit calendars; no entries about user visit frequency; no engagement dashboards | no | partial | Rule and adjacent disguises recover; intention-rotation layer is compressed away. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: "first visible frame feel already alive" with "No spinner" and quiet-field fallback | PLAN.md §§1,8,12: birds mid-action, bootstrap snapshot, quiet field, first-bird budget, no fade/static | no | full | Already-running first frame, snapshot mechanics, and spinner refusal recover. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: synthetic UUID avoids email as key/log/telemetry/URL | PLAN.md §§4,11: encrypted email, keyed fingerprint, email absent from identifiers/logs/telemetry/URLs | no | partial | PII leakage rationale recovers; hard-to-retrofit/compliance layer is not present. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md: tick preserves elapsed time and prior inputs, "not a frozen client resume" | PLAN.md §§3,6,7: once-per-minute tick, server only writer, clients render snapshots, multi-device canonicality | no | full | Tick, sync, and client-collapse rationale recover. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md: stale clients cannot overwrite state; client retries cannot duplicate events | PLAN.md §§5,7,14: additive server-authored deltas, append-only events, no absolute personality writes | no | full | Server deltas, lost-drift risk, and implementation invariant recover. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | RECONSTRUCTION.md: conflicts are "matter-of-fact" with no "naturalist jokes or charm" | PLAN.md §§5,7,11: system surfaces use matter-of-fact copy; errors need no charm | no | full | System clarity over charm in errors recovers. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: per-bird state stays inside simulation, not analytics/training/recommendations/population dashboards | PLAN.md §§4,11,12: event log for simulation only, aggregate telemetry only, simulation DB never read into analytics | no | full | Private relationship boundary and pipeline enforcement recover. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visits are read-only and visitors cannot affect host simulation | PLAN.md §§4,5,13: visitor snapshot only; no presence/offers/listen-in/settle/notebook events | no | full | Observation-not-co-presence rationale recovers. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: host visit log is "reachable on demand and never pushed" with no badge | PLAN.md §§2,4,5: visit notifications default false; log in settings; no badge/push/email by default | no | full | No attention-driver visit loop recovers. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | RECONSTRUCTION.md: no public social, discovery, leaderboards, or leaderboard-like stats | PLAN.md §§2,11,16: no leaderboards/discovery/public profiles; no computed ranking-like metrics | no | full | Public comparison/product-drift refusal recovers, though compactly. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: narration exposes "same aviary charm" without raw labels or event-log phrasing | PLAN.md §§10,13: naturalist prose from snapshot state, no raw mood/perch/personality values, AT testing | no | full | Running prose and same-product accessibility recover. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md: reduced motion is a "designed alternate visual register" not "broken static fallback" | PLAN.md §§8,10,13: cross-fades, no leaf drift, preserved calls/captions/narration/mood/identity | no | full | Alternate renderer, preserved core, and anti-stripped-fallback rationale recover. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md: hard budgets preserve the "already running" experience with first bird under 500ms | PLAN.md §§8,12,13: first bird under 500ms; performance metric as felt aliveness guard | no | full | Affective performance threshold recovers. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION.md: no achievements/streaks/counters and "No feature flag for gamification" | PLAN.md §§2,13,17: absolute no games/streaks/scores/badges; no feature flag; guardrails reject gamification words | no | partial | Absolute refusal and foothold guard survive; adjacent-product temptation layer is compressed. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md: excludes hunger/death/distress and preserves "non-punitive continuity" | PLAN.md §§2,6,16: no hunger/death/distress, no negative drift, observational not custodial | no | full | No obligation/punishment rationale recovers. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md: server-selected starters avoid a "catalog" and optimization/customization | PLAN.md §§2,4,5,16: system-selected starter birds, balanced variety, no catalog/customization model | no | full | Anti-avatar-catalog rationale recovers in optimization/customization terms. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md: arrival is "part of the aviary life, not a reward" and not based on engagement/payment | PLAN.md §§2,6,13,15: age-based only; not visit count, interactions, payments, or engagement | no | partial | Age-vs-reward layers recover; economy/unlock-management consequence is not explicit. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md: stable bird IDs preserve "persistent identity" across migrations/updates | PLAN.md §§4,7,16: bird_id never replaced; migrations preserve identity; sync correctness protects bird identity | no | partial | Identity continuity and distinction from stored vectors recover; retroactive-erasure consequence is missing. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md: mood persists across sessions and opening tab does not reset mood | PLAN.md §§6,7: mood persists, no neutral reset, no visible snapping to startup default | no | full | Continuity-through-mood rationale recovers. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md: notebook is read-only and not a log/achievement feed | PLAN.md §§4,5,6,8: no edit/delete endpoints, read-only forever, observer record not journal | no | full | Observer-record rationale recovers. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md: export is private data access, machine-readable, not product surface | PLAN.md §§2,4,5: JSON snapshot by verified email; not advertised or naturalist surface | no | full | Quiet copy/right-to-data rationale recovers. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md: deletion is "privacy and recovery"; hard delete removes simulation data | PLAN.md §§5,11,14: 30-day recovery, hard delete birds/vectors/notebook/events/invites/sessions/export jobs | no | full | Regret protection, privacy, and all-data deletion layers recover. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: aggregate-only operational telemetry without per-account/bird dimensions | PLAN.md §§11,12,14: allowed aggregate metrics; disallowed per-bird/per-account simulation fields | no | full | Technical telemetry boundary recovers. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md: invites are email-based, off by default, revocable, and non-public | PLAN.md §§4,5,13: per-email invites, no discoverability/public flag, revocation, no implicit sharing | no | full | Deliberate named sharing recovers. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md: host log gives "host control without attention pressure" | PLAN.md §§2,4,5: visit log in settings, no badge, no push/email by default | no | full | Transparency without social loop recovers. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | The snapshot rule is captured, but the no-show-off/real-birds rationale is not recovered. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: slow idle cadence, priority bumps, queue control, no announcement-style phrasing | PLAN.md §§10,14: 30-60 second cadence, polite live region, no flooding, observational updates | no | full | Slow rhythm, queue protection, and observational cadence recover. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | Y | Y | Y |
| F2 | Y | Y | Y |
| F3 | Y | Y | Y |
| F4 | Y | N | Y |
| F7 | Y | N | Y |
| F9 | Y | Y | N |
| F10 | Y | N | N |
| F12 | Y | Y | Y |
| F13 | Y | Y | Y |
| F14 | Y | N | Y |
| F15 | Y | Y | Y |
| F16 | Y | Y | N |
| F17 | Y | Y | Y |
| F18 | Y | Y | Y |
| F20 | Y | Y | Y |
| F24 | Y | Y | Y |
| F25 | Y | Y | Y |
| F27 | Y | N | Y |
| F30 | Y | Y | N |
| F31 | Y | Y | N |
| F35 | Y | Y | Y |
| F40 | Y | Y | Y |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From benchmark constants |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 49 | S whys always included; all F whys reachable here |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 127 / 152 | Weighted numerator over included whys |
| Whys with reconstruction evidence | 46 | Exact rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 47 | Exact PLAN grounding for rationale |
| rule_without_why cases | 3 | S4, F6, F39 |
| plan_only_not_reconstructed cases | 1 | S4 |
| ungrounded_reconstruction cases | 0 | None counted |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (full + partial-weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 weighted | 48.0 weighted | 88.9% |
| Affective whys | 98 weighted | 79.0 weighted | 80.6% |
| Weight-2 whys | 44 weighted | 39.0 weighted | 88.6% |
| Weight-3 whys | 108 weighted | 88.0 weighted | 81.5% |
| System-level whys | 28 weighted | 25.0 weighted | 89.3% |
| Feature-level whys (reachable) | 124 weighted | 102.0 weighted | 82.3% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered better (48/54, 88.9%) than affective whys (79/98, 80.6%). The candidate carried architectural constraints very strongly: presence, server tick, privacy boundaries, and telemetry were all clear. The thinner spots were product-feel rationales like S2, S4, F10, F14, F27, F30, F31, and F39.
- **Weight-3 vs weight-2.** Weight-2 whys recovered at 88.6%; weight-3 whys at 81.5%. The harder multi-layer whys exposed compression: primary rules survived while secondary temptations or downstream consequences often dropped.
- **System-level vs feature-level.** System fidelity was higher (89.3%) than feature-level fidelity (82.3%). The plan encoded philosophy broadly, but feature-specific exceptions sometimes became mere rules.
- **Multi-layer recovery patterns.** The most commonly dropped layer was L2 or L3: F4 missed chorus phase-cancel rationale, F9 missed the generic arrival-animation failure, F10 missed the reframing consequence, F16 missed retrofit/compliance, F30 missed unlock-economy collapse, and F31 missed retroactive evaporation of presence-time.
- **Subdomain patterns.** Accounts/sync, privacy, and accessibility were strong. Audio and social were mixed: procedural audio survived, but the chorus-specific rationale thinned; visits were operationally captured, but F39's no-show-off rationale was absent.
- **Evidence-bound effects.** F6 and F39 are the cleanest rule-without-why cases. S4 was also evidence-bound: the plan implements restraint widely, but the frozen reconstruction does not identify restraint-over-richness as the governing why.

What this suggests: the candidate is excellent at preserving implementation invariants and broad product boundaries, but it compresses some deeper affective anti-pattern rationales into generic no-go rules.

---

## 4. Recommendations for v2 hardening

- Keep v06 evidence-bound scoring. It cleanly distinguished rules from rationales and prevented F6/F39 from getting credit for merely naming the feature.
- Add more targeted affective exceptions like F29-F40. This run is near-saturated on planning coverage; discrimination came from non-obvious rationale rather than feature presence.
- Consider adding an instruction to phase-1 planners to preserve tempting anti-patterns and why they are wrong. That would test whether the model can encode rationale without being handed the gold IDs.
- Preserve the system-level cross-cutting test, but continue auditing it carefully. S4 demonstrates why a plan can be operationally cross-cutting while a reconstruction still misses the named principle.
- Track layer-level diagnostics across waves. Here, primary mechanics survived more reliably than secondary/downstream consequences; if that pattern repeats, v2 should weight anti-pattern/failure-mode preservation more explicitly.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The prompt stated this was fresh phase-2B context. I did not modify the frozen reconstruction, read PRD files, read peer slots, or use prior run memory.
- **Single-run limitation.** This is one pass for this candidate/evaluator setup; it has no variance signal by itself.
- **Borderline capture calls.** Feature 7 and feature 87 were marked missed. Feature 97 was counted captured operationally because visitor snapshots imply the host aviary, but its why was scored none.
- **System-level cross-cutting.** S4 is the most subjective call: the plan preserves restraint through many decisions, but the reconstruction does not identify the governing principle. It therefore receives partial via PLAN cross-cutting only.
- **Confabulation cases.** None counted. The reconstruction reads as plan-derived; no ungrounded rationale was credited.
- **Evidence-bound denials.** F6 and F39 were denied for missing rationale evidence. Several multi-layer rows received partial credit because only some layers met the evidence gate.
- **Rule-without-why cases.** S4, F6, and F39 preserved mechanisms or constraints without the full gold rationale.

---

End of report.
