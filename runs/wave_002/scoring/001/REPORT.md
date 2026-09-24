# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Scored from the frozen reconstruction only; PLAN evidence establishes capture/grounding but not feature-level recovery by itself.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **99.2%** |
| Intent fidelity | **62.5%** |
| Combined quality | **9879** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **58.1%**

**(Planning, fidelity) coordinate:** `(99.2, 62.5)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-24T07:16:22Z |
| Candidate model | gpt-6-sol |
| Candidate effort | extra-high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **119 / 120** = **99.2%**.

By PRD file:

| File | Total | Captured | Rate |
| --- | --- | --- | --- |
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 17 | 94.4% |
| accounts_sync.md | 18 | 18 | 100.0% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| Total | 120 | 119 | 99.2% |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
| --- | --- | --- | --- | --- |
| 1 | Headline product concept statement | product_brief.md | yes | PLAN.md:5-7 product contract, tone, no-gamification, and scope refusals. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | PLAN.md:5-7 product contract, tone, no-gamification, and scope refusals. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | PLAN.md:5-7 product contract, tone, no-gamification, and scope refusals. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | PLAN.md:5-7 product contract, tone, no-gamification, and scope refusals. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | PLAN.md:5-7 product contract, tone, no-gamification, and scope refusals. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | PLAN.md:5-7 product contract, tone, no-gamification, and scope refusals. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | PLAN.md:54,60,62 define presence, drift, mood, and settle behavior. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | PLAN.md:54,60,62 define presence, drift, mood, and settle behavior. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | PLAN.md:54,60,62 define presence, drift, mood, and settle behavior. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | PLAN.md:54,60,62 define presence, drift, mood, and settle behavior. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | borderline; Borderline inclusive: PLAN.md:26 and 60 include vocal_frequency in the vector/drift model, and PLAN.md:64/82 schedule and synth calls from descriptors. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | PLAN.md:26,58,60,62,64,66,68 cover bird state, drift, mood, calls, greeting, and adoption. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 34 | Greeting variation by absence length | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | PLAN.md:54,56,66,78,82 cover presence, listen-in, offers, settle, notebook, and no visit-frequency surfaces. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Missed: plan covers day/night and quiet field states but not the calm naturalist palette or saturated-accent exclusion. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | PLAN.md:9,76,78,80 cover scene, perches, motion, loading, top bar, and reduced motion. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | PLAN.md:23-33,41-48,58,72,96 cover accounts, sync, export/deletion, tick, and telemetry. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | PLAN.md:9,30,47-50 cover per-invite read-only visits, revocation, logs, and social refusals. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | PLAN.md:80,88,90,94,96 cover narration, reduced motion, captions, performance, and observability. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | PLAN.md:7 and 117-120-equivalent non-goal language in the plan contract. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | PLAN.md:7 and 117-120-equivalent non-goal language in the plan contract. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | PLAN.md:7 and 117-120-equivalent non-goal language in the plan contract. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | PLAN.md:7 and 117-120-equivalent non-goal language in the plan contract. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:5 "continuing canonical world"; 15 "first frame should feel ongoing" | PLAN.md:58 server tick; 64 procedural calls; 76 ongoing first frame/no spinner | yes | yes | no | partial | Continuing-world and cross-stack aliveness survived; downstream staleness/leaving consequence was not reconstructed. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md:61 "Bird greetings carry the welcome"; 67 visit email off by default | PLAN.md:7 no return toast/status badge; 50 default-off visit email; 66 notebook avoids user-frequency statements | yes | yes | no | partial | The no-announcement rule survived, but the seen-vs-processed affective rationale and cumulative-toast hazard did not. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md:7 "quiet, naturalist, and non-gamified"; notebook/narration avoid stats | PLAN.md:7 naturalist observations; 66 bird/place observations; 88 naturalist narration | yes | yes | no | full | Specific naturalist observation over generic metrics is reconstructed and cross-cutting. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md:5 two birds/max seven/one scene; 76 one viewport; 78 sparse top bar; 82 chorus recognizability | no | yes | yes | partial | The plan preserves restraint in several decisions, but the reconstruction never names depth-over-variety as a system principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md:59 naturalist observations and plain system language; 243 direct system errors | PLAN.md:7 naturalist product/system language split; 90 errors/settings use plain language | yes | yes | no | full | The product/system voice split is explicitly reconstructed and preserved across account, error, narration, and notebook surfaces. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:11 qualified attention; 151 visible/focused/recent activity; 9 absence not punished | PLAN.md:54 presence conjunction and settle/tab-close equivalence; 56 no surveillance; 60 presence-drift input | yes | yes | no | full | The reconstruction carries precise attention, false-presence rejection, and no-punishment consequences. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:5 database sole source/simulation only writer; 157 due minute with no clients; 177 one world timeline | PLAN.md:15 simulation worker only writer; 58 scheduled server tick; 72 devices pull canonical projection | yes | yes | no | full | Canonical server tick, multi-device coherence, and lost/duplicated drift avoidance are recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md:17 privacy-minimal operations; 253 aggregate only; 255 no telemetry/model-training access | PLAN.md:27 interaction events never feed analytics; 96 no relationship data in telemetry/model pipelines | yes | yes | no | full | The private relationship data boundary is reconstructed at the data-pipeline level. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:13 same experience not fallback; 57 meaningful with SR/reduced-motion/audio off; 279 not static fallback | PLAN.md:5 ships narration/reduced motion/captions in v1; 80 same state/calls/notebook; 88-90 SR and keyboard acceptance | yes | yes | no | full | Accessibility is recovered as same-product experience, designed mode, and release-gated v1 surface. |

Multi-layer system whys:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| S1 | yes | yes | no |
| S2 | yes | no | no |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: PLAN.md:58 scheduled tick; 64 procedural calls; 76 ongoing first frame/quiet field; 84 no fake loops.
- S2: PLAN.md:7 no return toast/status badge/notifications; 50 visit notifications default off; 66 notebook avoids user-frequency; 78 no scene badge/control.
- S3: PLAN.md:7 specific naturalist observations; 66 bird/place notebook observations; 88 naturalist narration; 102 copy reviews reject generic/gamification language.
- S4: PLAN.md:5 two birds/max seven/one scene; 76 one viewport; 78 sparse top bar; 82 chorus recognizability and no track UI.
- S5: PLAN.md:7 product/system language split; 45 matter-of-fact 409; 50 unavailable visit surface; 90 direct system/accessibility language.
- S6: PLAN.md:54 presence conjunction and settle/tab-close equivalence; 56 overlap union/no surveillance; 60 drift input; 102 presence tests.
- S7: PLAN.md:15 simulation worker only writer; 58 scheduled server tick; 72 canonical projection; 96 tick lag observability.
- S8: PLAN.md:27 events never feed analytics; 33 PII/retention boundary; 96 aggregate-only telemetry/model-training exclusion; 111 privacy leakage controls.
- S9: PLAN.md:5 ships accessibility surfaces in v1; 80 reduced motion same state/calls; 88 narration queue; 90 keyboard/contrast/testing; 110 accessibility release gates.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **58.1%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md:151 visible/focused/recent pointer-key; rejects open tabs and offline backfill | PLAN.md:54 all three conditions; 56 no open-tab/offline backfill; 60 drift input | no | partial | Precise conjunction survives; individual-signal failure cases and silent population corruption are not reconstructed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:161 slow low-pass signal; 271 synthetic week/three-week trajectories | PLAN.md:60 low-pass signal, week-one numeric and roughly three-week perceptible change | no | partial | Slow low-pass calibration survives; Tamagotchi-vs-screensaver failure band is not recovered. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:9 absence cannot harm/reset; 161 nonnegative deltas; 169 quieter without distress | PLAN.md:7 absence cannot harm; 60 delta >= 0; 62 absence quieter without distress | no | full | No negative drift, no absence punishment, and quieter-not-distressed return all survive. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:35 grammar descriptors/seeds/signatures; 221 WebAudio synth voices; 227 avoids fake loops | PLAN.md:64 current/upcoming descriptors and no recorded loops; 82 WebAudio synth voices; 84 silence/captions fallback | no | partial | Procedural no-loop and WebAudio cascade survive; chorus phase-artifact rationale is not reconstructed. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:37/213 and PLAN.md:78, but not the why that users read mood from motion without labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md:179 voice limits preserve recognizability through seven birds | PLAN.md:64 bound density and simultaneous voices to preserve recognizability through seven birds | no | full | The cap is grounded in per-bird audio recognizability, though the reconstruction is compressed. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md:109 immutable UUID and persisted vector; 265 protect existing identities and vectors | PLAN.md:15 clients cannot set traits; 26 current persisted vector; 58 reset cannot be hidden | no | partial | Canonical server persistence and downstream sync rules survive; losing the known bird is not articulated. |
| F8 | personality-vector-never-numerical | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:63/97 and PLAN.md:17/42/88, but the stat-management relationship rationale is absent. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md:183 current state plus absence duration; response to continuing aviary | PLAN.md:66 one primary bird, absence/boldness/mood variation, per-arrival seed | no | partial | Absence/current-state variation and not-waking-the-app survive; one-bird/procedural anchor is thin in reconstruction. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md:61 bird greetings carry welcome; avoids metric panels, guilt, prompts | PLAN.md:7 bird greetings carry welcome and no return toast/absence counter/status badge | no | partial | The rule and adjacent no-counter extensions survive; seen-vs-announced rationale does not. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:47 and PLAN.md:54, but the chore/punishment rationale is not reconstructed. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md:49 rare immutable naturalist observations; 185 sparse, naturalist, avoid visit-frequency/trait statements | PLAN.md:66 naturalist grammar, rarity threshold, multi-day cooldown, immutable sparse entries | no | partial | Naturalist specificity and rare/read-only shape survive; stock event-log spell-breaking rationale is absent. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md:151 qualifies actual attention and rejects open tabs/offline backfill | PLAN.md:54 visible/focused/recent activity; 56 no open tab or unverified backfill | no | partial | Conjunction precision survives; per-signal misses and silent population-wide drift corruption are not reconstructed. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md:61 avoids metric panels/guilt/prompts; 185 never visit-frequency statements | PLAN.md:7 no absence counter/status badge; 66 notebook never user visit-frequency; 102 copy rejects gamification vocabulary | no | partial | No streak/no user-frequency surfaces survive; the intention-rotation into number management is missing. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md:15 first frame ongoing/no spinner/no reset; 205 no spinner/static fade; 207 quiet field | PLAN.md:76 phase anchors, no spinner/fade, soft quiet field; 13 private edge snapshot | no | full | Ongoing first frame, server snapshot mechanics, and quiet-field-not-spinner all survive. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md:103 synthetic UUID/no email-derived keys prevents email becoming system-wide identifier | PLAN.md:23 synthetic UUID primary key and no email-derived keys; 33 UUIDs across rest of system | no | partial | Synthetic UUID and PII-spread prevention survive; retrofit/impossible-later consequence is missing. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md:157 due minute even with no clients; 177 two devices consume one world timeline | PLAN.md:58 scheduler queues due minute with no clients; 72 devices pull same canonical projection | no | full | Server tick, multi-device coherence, and divergent-client avoidance are all substantially recovered. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md:133 no absolute personality/mood writes; 273 prevent lost or duplicated drift | PLAN.md:43 no absolute writes; 58 additive server-authored deltas; 72 personality has no UI conflict | no | full | Additive server-authored deltas, lost-drift risk, and no client personality mutation survive. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | RECONSTRUCTION.md:59 errors/account surfaces stay direct; 243 never evasive bird prose | PLAN.md:45 matter-of-fact 409; 90 system errors and accessibility settings use plain direct language | no | full | The error-context exception to naturalist voice is directly reconstructed. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md:111 interaction events never feed analytics; 255 no telemetry/model-training access | PLAN.md:27 events never feed analytics; 96 simulation database/private events inaccessible to telemetry/model-training | no | full | Simulation-only use, relationship-data privacy, and pipeline boundary all survive. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md:51 visitor watches same scene without controls so visiting cannot change aviary | PLAN.md:9 same scene without host controls; 50 no owner endpoints; 56 visitors have no presence record | no | full | Read-only observation and no visitor drift are recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:67/141 and PLAN.md:50, but not the attention-driver/engagement-loop rationale. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | none | none | yes | none | RECONSTRUCTION.md:65 marks the no-public-discovery cluster NOT RECOVERABLE FROM PLAN; score none per instruction. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:231 same world not parallel state dump; 233 naturalist narration; 235 no raw labels | PLAN.md:88 naturalist prose and no state mutations; 90 real assistive-technology testing | no | full | Naturalist running prose, same-product experience, and not-ARIA-state-dump implementation survive. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md:219 still-pose cross-fades and same canonical state; 279 not static fallback | PLAN.md:80 cross-fades, no parallax, same state/calls/notebook/moods/drift | no | full | Different rendering, same aviary, and non-stripped fallback all survive. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:249 and PLAN.md:94, but the affective performance threshold rationale is missing. |
| F27 | no-gamification | 4 | yes | included | none | none | yes | none | RECONSTRUCTION.md:65 marks the gamification non-goal cluster NOT RECOVERABLE FROM PLAN; score none per instruction. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md:9 absence cannot harm/reset; 169 quieter expression without distress | PLAN.md:7 absence cannot harm birds; 60 no negative drift; 62 no distress | no | full | No suffering, no neglect punishment, and observational rather than custodial relation survive. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | RECONSTRUCTION.md:27 marks system-selected starter birds NOT RECOVERABLE FROM PLAN; score none. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md:187 age alone, no badges/urgency/catalog/reward; calm age milestones | PLAN.md:68 opportunities from aviary age alone, never session/visit/offer totals, no catalog/rarity draw | no | partial | Age-not-attention reward refusal survives; economy/erosion downstream consequence is missing. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md:29 renames preserve identity/continuity; 265 rollback without regenerating birds | PLAN.md:15 migrations preserve stable IDs/vectors; 26 immutable UUID; 98 never regenerate birds | no | full | Same individual across rename/migration, identity beyond vector storage, and no reset/regeneration all survive. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md:167 persisted finite state without resetting on page open; 283 elapsed-time mood | PLAN.md:62 mood pull occurs in ticks, not page open; no session-start reset | no | full | Mood continuity across sessions and no neutral reset are recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:49/115 and PLAN.md:66, but not the observer-record-not-journal rationale. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:53/97 and PLAN.md:17/33, but not the quiet relationship-copy rationale. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md:53 deletion removes linked state after recovery; 121 retention limited to recovery/deletion needs | PLAN.md:33 recoverable for 30 days, then delete birds/vectors/events/notebook/sessions/invitations/visits/exports | no | partial | Grace recovery and full deletion scope survive; privacy-claim rationale for hard delete is thin. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md:253 aggregate counters only; 255 no telemetry/model-training access; 257 no relationship data | PLAN.md:96 aggregate RUM only, no account/bird/event dimensions, no relationship data | no | full | The technical observability boundary against relationship-data back doors is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md:117 encrypted recipient address, one-use token, revocation; 141 host controls access/history | PLAN.md:30 encrypted recipient address and scoped grant; 47 host-only invitations; 50 one-use scoped visit grant | no | full | Per-invite deliberate sharing and no ambient discovery/control loss are recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:67/141 and PLAN.md:47/50, but not transparency-without-social-loop rationale. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Rule appears at RECONSTRUCTION.md:51/73/99 and PLAN.md:9, but not the actual-witnessing/no-show-off rationale. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md:233 deduplicated idle cadence with polite preemption; 231 not a parallel state dump | PLAN.md:88 idle cadence 30-60 seconds, polite preemption, no list of state mutations | no | partial | Sparse observational cadence and user-event priority survive; screen-reader queue overwhelm rationale is absent. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| F1 | yes | no | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | no | yes |
| F7 | yes | no | yes |
| F9 | no | yes | yes |
| F10 | yes | no | yes |
| F12 | yes | no | yes |
| F13 | yes | no | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | no | no | no |
| F30 | yes | yes | no |
| F31 | yes | yes | yes |
| F35 | yes | no | yes |
| F40 | yes | no | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
| --- | --- | --- |
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Fixed full-instance possible weight |
| Reachable gold whys | 49 | All 40 feature whys reachable; system whys always included |
| Excluded unreachable feature whys | 0 | No feature-why anchors excluded |
| Recovered / reachable weight | 95.0 / 152.0 | Weighted numerator and denominator |
| Whys with reconstruction evidence | 36 | Rows with non-none rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 37 | Rows with non-none plan rationale/cross-cutting grounding |
| rule_without_why cases | 13 | Mechanism/rule survived without the gold rationale |
| plan_only_not_reconstructed cases | 1 | S4 cross-cutting restraint preserved in PLAN but not named by reconstruction |
| ungrounded_reconstruction cases | 0 | No clear ungrounded rationale assertions counted |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weighted credit | Recovery rate |
| --- | --- | --- | --- |
| Functional whys | 54.0 | 38.0 | 70.4% |
| Affective whys | 98.0 | 57.0 | 58.2% |
| Weight-2 whys | 44.0 | 21.0 | 47.7% |
| Weight-3 whys | 108.0 | 74.0 | 68.5% |
| System-level whys | 28.0 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 124.0 | 72.0 | 58.1% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered **38.0 / 54.0** weighted credit (70.4%), while affective whys recovered **57.0 / 98.0** (58.2%). The reconstruction is strongest where the plan encodes hard architecture: S6, S7, F18, F20, F25, and F36.
- **Weight-3 vs weight-2.** Weight-3 whys recovered **74.0 / 108.0** (68.5%), better than weight-2 whys at **21.0 / 44.0** (47.7%). High-weight architectural invariants survived especially well, but high-weight affective exclusions such as S2, F9, F10, F14, and F30 often lost downstream consequence layers.
- **System-level vs feature-level.** System-level fidelity (**82.1%**) is much higher than feature-level fidelity (**58.1%**). The plan carried the philosophy broadly, but the reconstruction compressed many feature whys into operational rules.
- **Multi-layer recovery.** Primary causes were most likely to survive. Secondary and downstream layers dropped when they were affective consequences rather than implementation failure modes: examples include S1, S2, F4, F9, F10, F14, and F40.
- **Subdomain patterns.** Accounts/sync, privacy, server simulation, and accessibility did well. Social-option rationales were weakest: F22, F38, and F39 preserved rules but not why those rules protect the relationship and avoid attention loops.
- **Evidence-bound effects.** v06 denied credit for several plausible semantic recoveries because the frozen reconstruction gave only the mechanism: F5, F8, F11, F22, F26, F33, F34, F38, and F39. F23, F27, and F29 explicitly said `NOT RECOVERABLE FROM PLAN`, so they score none.

What this suggests: the candidate plan is very broad and operationally strong, but still compresses several product-feel justifications into policy-like exclusions. The reconstruction mirrors that compression.

---

## 4. Recommendations for v2 hardening

- Keep the targeted F29-F40 headroom style. In this run those rows separated high planning coverage from rationale recovery, especially for starter birds, visit logs, actual-aviary visiting, account export, and notebook read-only behavior.
- Add more feature-level exception whys in quiet social/privacy areas. The candidate captured the social rules, but the reconstruction often lost why attention loops and show-off surfaces would change the product.
- Preserve multi-layer whys for affective rules, not just architecture. The architecture layers are recoverable from implementation detail; affective downstream consequences need explicit plan language to survive.
- Consider a separate diagnostic for plan-only cross-cutting principles. S4 was preserved by the plan's rules but not named by the reconstruction, producing a partial system score that is methodologically valid but subjective.
- Keep the evidence-bound operator. It made the most important distinction in this run: mechanisms survived more often than reasons.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction reads plan-derived and the validity audit verdict is PASS. No same-context evidence appeared.
- **Single-run limitation.** This is one run; no variance estimate is available.
- **Borderline capture calls.** Feature 20 was counted captured inclusively because the plan carries `vocal_frequency` through the vector/drift model and call scheduling, but it does not state the exact call-timing rule as plainly as the gold list. Feature 69 was the only planning miss.
- **System-level cross-cutting.** S4 is the subjective case: restraint is clearly cross-cutting in the plan, but the frozen reconstruction does not name it as a system-level principle.
- **Confabulation cases.** No clear ungrounded reconstruction cases were counted.
- **Evidence-bound denials.** Rule-only rows are the main v06 effect: 13 included whys were marked `rule_without_why`.
- **Operational compromise.** TIMING.json had phase 1 and phase 2A timing for run 001 but no phase 2B timing. The score JSON includes only the available timing fields.

---

End of report.
