# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Evidence for every S1-S9 and F1-F40 row is taken from the frozen reconstruction and assigned PLAN only.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **93.3%** |
| Intent fidelity | **62.3%** |
| Combined quality | **9296** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **57.6%**

**(Planning, fidelity) coordinate:** `(93.3, 62.3)` - plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-06-30T21:05:10Z |
| Candidate model | claude-5-sonnet |
| Candidate effort | medium |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **112 / 120** = **93.3%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 17 | 85.0% |
| aviary_layout.md | 18 | 16 | 88.9% |
| accounts_sync.md | 18 | 15 | 83.3% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **112** | **93.3%** |

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline: no glossary section, but the plan defines the domain terms through data-model and engine sections. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Borderline: personality-shaped call parameters imply individual signatures, but recognizability by ear is compressed. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | Missed: plan relies on return-greeting but never explicitly forbids a textual welcome toast/banner. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Missed: offer events/reactions appear, but mood-and-curiosity variation of the reaction is not specified. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | Missed: no per-bird offer cooldown is specified. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Borderline: server-chosen perch zones imply no user placement, but the refusal is not named. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Borderline: night/day and nightjar class appear, but the full night-state behavior is compressed. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Borderline: top-bar affordances are spread across account/accessibility/notebook/offer sections rather than listed once. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | Missed: top bar is present, but idle auto-fade behavior is absent. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | Missed: no empty-aviary transitional state is described. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Borderline: calm/naturalist palette is implied through quiet field and scene palette work, not fully specified. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | Missed: magic links are single-use/short TTL, but the 15-minute expiry is not carried. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | Missed: the plan prevents sync conflicts but does not specify a sync-conflict/account-error tone surface. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Missed: no privacy-policy link in account settings is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Borderline: visits are opt-in/per-invite and notifications default false, which carries default-off semantics. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Borderline: visitors read the host snapshot/audio, enough to imply actual aviary, but no-show-off mode is not named. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION §System intent: "The aviary continues without the viewer"; "repetition breaks the premise." | PLAN §4.1 tick runs with no client; §8.2 mid-pose/quiet field; §9.1 calls not byte-identical. | yes | yes | no | full | All three aliveness layers survive across simulation, loading, greeting/audio, and motion. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION §System intent: "Growth and interaction must avoid gamification" and social is "intentionally narrow". | PLAN §1.2 bans gamification/push; §3.1 visit_notifications_enabled default false; §10.1 avoids announcement-like narration. | no | yes | no | partial | PLAN preserves the refusal across multiple surfaces, but B does not articulate the noticed-vs-announced rationale. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION §System intent: "Ambient naturalist product voice"; §12 "specificity in bird names, timing, and behavior." | PLAN §12.1 naturalist templates with names/specifics; §10.1/10.3 narration/captions share the same voice system. | yes | yes | no | full | Specific naturalist prose and anti-generic voice survived strongly. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION §Scope: 7-bird cap and no broad social/custom/payment surfaces, but no depth-over-variety rationale. | PLAN §1.2 excludes custom scenes/social/payment; §8.1 single scene; §1.3 age-paced growth to max 7. | no | yes | no | partial | Rules of restraint survive, but the per-bird-recognizability/depth rationale is not reconstructed. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION §System intent: "Ambient naturalist product voice over system-surface intrusion"; §5 matter-of-fact unavailable surface. | PLAN §10.1/§12.1 naturalist prose; §5.5 matter-of-fact visit unavailable; §11.6 matter-of-fact unsupported-browser page. | yes | yes | no | full | The voice split is compressed but present: naturalist for aviary surfaces, matter-of-fact for system/error surfaces. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION §4: "Presence pings only while visible, focused, and recently active" and settle closes presence cleanly. | PLAN §4.4 all three conditions; §4.3 presence-time dominant; §8.7 settle ends presence like tab-close. | yes | yes | no | partial | Idle attention and settle equivalence survive, but the population-inflation failure mode is mostly absent. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION §System intent: "Server-authoritative life, not client-authored state"; "client never simulates." | PLAN §2.1 simulation service runs without clients; §6.1 one canonical snapshot; §6.4 only tick writes personality. | yes | yes | no | full | Server tick, sync coherence, and no-last-write-wins rationale are all recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION §System intent: "Privacy boundaries are load-bearing" and telemetry is "aggregate-only." | PLAN §7.2 analytics never reads simulation DB; telemetry SDK rejects per-bird/per-account fields. | yes | yes | no | full | The technical data-pipeline boundary and relationship-data privacy stance survive. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION §System intent: accessibility is "a designed v1 surface, not a fallback." | PLAN §13.1 accessibility ships with v1; §8.6 reduced-motion authored alongside full motion; §10 narration/captions/keyboard. | yes | yes | no | full | Designed accessible charm, launch gating, and non-fallback implementation survive. |

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | no |
| S6 | yes | no | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: server tick without clients; scene loads mid-motion/quiet field; procedural calls; reduced-motion charm; narration/captions.
- S2: no gamification/push; visit notifications default false; polite narration; no visit-frequency notebook inputs; age-based growth.
- S3: naturalist notebook prose; naturalist narration/captions; bird names in prose; generic phrasing rejected; no public comparison surfaces.
- S4: two starters/max seven; single scene; no custom scenes; no broad social surfaces; compact top-bar pattern.
- S5: naturalist notebook/narration/captions; matter-of-fact visit unavailable and unsupported-browser pages; account/system surfaces separate from aviary voice.
- S6: presence pings require visibility/focus/activity; presence dominates drift; settle equals tab-close at engine level; no streak/counter surfaces.
- S7: server-side tick; client never owns state; one canonical snapshot for sync; no last-write-wins; event log is append-only.
- S8: synthetic UUIDs; separate simulation DB/telemetry; no ML pipeline; export/deletion; aggregate-only observability.
- S9: naturalist narration; reduced-motion render mode; captions; keyboard/focus; accessibility ships with v1.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **57.6%**.

Reachable feature-level whys: **38 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION §4: presence pings only while visible, focused, recently active. | PLAN §4.4: visible + focus + recent pointer/key activity; §4.3 presence-time dominates drift. | no | partial | L1 survives; laxer-tab shortcut and silent population-wide corruption are not fully reconstructed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION §14: too-fast drift becomes Tamagotchi, too-slow becomes screensaver; §4 calls drift calibration a CI fixture. | PLAN §4.3: low-pass filter, one-week instrument / three-week visible calibration, too-fast/Tamagotchi and too-slow/screensaver risks. | no | full | All calibration layers survive. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION §4: non-negative deltas are load-bearing; §1 excludes Tamagotchi mechanics and negative drift. | PLAN §4.3: deltas are never negative; sign error is damaging; §1.2 excludes hunger/death/decay. | no | partial | No-negative and no-Tamagotchi survive; the quieter-not-mistrust relationship consequence is thin. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION §9: WebAudio procedural synthesis with jitter; no recorded-audio fallback path; true chorus. | PLAN §4.6/§9.1: motif grammar plus micro-variation; §9.2 true chorus; §9.4 no recorded-audio pipeline. | no | full | Procedural audio rationale is one of the strongest recoveries. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION §8: mood drives perch and pose; client reads mood directly. | PLAN §8.3: mood-shaped pose-blend tables and perch-zone selection. | yes | none | The mechanism survives, but not the why that users should read mood without labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION §Scope: 7-bird cap tied to age-paced growth. | PLAN §1.1/§1.3: server-controlled growth to 7-bird cap. | yes | none | The cap survives, but not the recognizability-by-ear rationale. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION §System intent: server-authoritative state; §6 no last-write-wins for personality. | PLAN §3.2 personality vector persisted server-side; §6.4 only tick writes personality. | no | partial | Canonical persistence and sync implications survive; losing-the-bird affective rationale is absent. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION §3: user perceives traits through mood, perch, and call parameters rather than numeric personality values. | PLAN §3.2/§4.5: snapshot omits personality; no browser-reachable API exposes it. | no | full | The anti-stat-management rationale is compressed but semantically present. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION §5: return-greeting encoded in snapshot using absence length, boldness/mood, and descriptors. | PLAN §5.1: greeting_event computed from fresh presence window, absence length, boldness/mood, randomized stagger. | yes | partial | Variation mechanics survive; the notice-never-announce downstream consequence is mostly missing. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured in PLAN, so this why is excluded from the denominator. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION §8: settle ends presence without becoming a different engine state. | PLAN §8.7: settle ends presence the same way tab-close does; no drift weighting. | yes | none | The engine equivalence survives, but not the chore/penalty rationale. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION §12: naturalist prose with bird names, timing, behavior; sparse; never generic. | PLAN §12.1/§12.2: naturalist templates, noteworthy moments, one entry every few days, no generic phrasing. | no | full | Naturalist specificity, voice concentration, and sparsity all survive. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION §4: presence pings only while visible/focused/recently active; no grace period. | PLAN §4.4: all three conditions simultaneously, server accumulation, ping density not rounded up. | no | partial | Precise rule survives; the examples of individual signals failing and silent drift corruption are mostly absent. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION §1/§14: no achievements, streaks, scores, badges, visit calendars; no dormant scaffolds. | PLAN §1.2 and §14.5: gamification primitives must not exist; no visit-frequency aggregates in notebook/telemetry. | no | partial | The structural anti-gamification refusal survives; the full presence-for-birds-not-counter rationale is partial. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION §8: first snapshot rendered mid-pose; quiet-field loading mode instead of spinner. | PLAN §8.2: HTML inlines first snapshot; birds render mid-pose; quiet field with faint motion, never spinner. | no | full | Loading and first-frame aliveness are recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION §7: synthetic account ID everywhere; email in exactly one encrypted column; impossible to retrofit. | PLAN §7.1: every table/log/queue/telemetry key uses account_id, never email; CI guard catches email fields. | no | partial | Identifier rule and retrofit consequence survive; explicit PII/log-leak rationale is thin. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION §1: server-authoritative life; §4 tick runs with no connected client; §6 canonical snapshot. | PLAN §4.1 tick reads events, updates canonical state, runs without clients; §6.1 sync reads single snapshot. | no | full | Server tick, multi-device coherence, and client-collapse failure avoidance survive. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION §6: only tick writes personality; request schemas have no personality-shaped field. | PLAN §6.4: only tick writes under row lock; API has no personality value path; §4.3 server-authored deltas. | no | partial | Implementation rule survives; overwrite-loss scenario is not reconstructed. |
| F19 | sync-conflict-tone | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured in PLAN, so this why is excluded from the denominator. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION §7: analytics never read simulation DB; per-bird/per-account data never leaves boundary. | PLAN §7.2: simulation DB separate from telemetry; no ML/training pipeline; SDK rejects relationship fields. | no | full | All privacy/data-pipeline layers survive. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION §11: visitors cannot write events and visitor attention never drifts host birds. | PLAN §5.5: visitor sessions are read-only snapshot/audio and POST /aviary/events rejects visitor tokens. | no | full | Read-only observation vs co-presence survives. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION §3.1/§Scope: visit_notifications_enabled default false and push notifications out of scope. | PLAN §3.1 visit_notifications_enabled default false; §1.2 excludes push notifications. | yes | none | The default-off mechanism survives, but not the attention-driver rationale. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION §11: no profiles, follows, feeds, comments, co-presence, shared cursor, or chat. | PLAN §1.2 excludes profiles, follows, public feed, discovery, comments; §14.5 no gamification tables. | yes | none | Public/comparison surfaces are refused, but the theirs-vs-compared rationale is absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION §10: live naturalist prose from snapshot; same voice as notebook; not differently voiced products. | PLAN §10.1: naturalist prose in aria-live region; shares notebook voice; prioritized events remain observations. | no | partial | Naturalist prose and voice continuity survive; ARIA/state-list failure mode is partial. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION §8: reduced-motion parallel render mode avoids stripped-fallback failure. | PLAN §8.6: curated still poses, cross-fades, authored alongside full motion; §13.1 ships with v1. | no | partial | Different rendering and non-fallback consequence survive; some same-aviary details are compressed. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION §11: first bird under 500ms via inlined snapshot before non-critical code. | PLAN §11.2: edge-inlined snapshot, critical render before audio/settings/notebook code, CI gate. | no | full | Performance as felt aliveness is adequately carried. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION §14: no dormant achievements or gamification tables; make creep structurally hard. | PLAN §1.2 and §14.5: no achievements/streaks/scores/badges/visit calendars; no scaffolds or aggregates. | no | full | The anti-gamification rationale and structural hardening are fully recovered. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION §1: no death, hunger, decaying happiness, distress, or negative drift. | PLAN §1.2 excludes Tamagotchi mechanics; §4.3 deltas never negative. | yes | none | The rule survives, but not the observational-not-custodial/absence-punishment rationale. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION §13: starts at two server-selected species, not user-chosen. | PLAN §13.2: two server-selected species, not user-chosen. | yes | none | Server selection survives, but meeting-arrivals-not-configuring-avatars does not. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION §13: age-driven scheduled checks; never engagement-driven; prevent interact-more-to-unlock. | PLAN §1.3/§13.2: unlocks based on aviary age, not engagement; config schedule; scheduled checks. | no | partial | Rejecting attention-score rewards survives; relationship-deepening phrasing is thin. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION §3: Bird id is stable and never reassigned. | PLAN §3.2: Bird id stable identity, never reassigned. | yes | none | Stable ID rule survives, but not the remembered-relationship rationale. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION §4: mood persists across sessions and does not reset on tab open. | PLAN §4.2: mood persists across sessions/ticks; no tab-open reset. | no | full | Mood continuity rationale is recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION §3: notebook read-only with no edit/delete API, not just hidden. | PLAN §3.6/§5.3: no write/edit/delete API exists. | yes | none | Read-only mechanism survives, but not the observer-record-vs-journal rationale. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION §7: export includes current personality values and moods; account control. | PLAN §7.3: async export JSON of birds, personality, moods, notebook, settings. | yes | none | Export substance survives, but the quiet relationship-copy rationale is not reconstructed. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION §1/§7: 30-day changed-mind window and cascading hard deletion including telemetry refs. | PLAN §5.4/§7.3: pending deletion, undelete within 30 days, then cascading hard delete. | no | full | Regret window, privacy hard-delete, and complete cascade survive. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION §7: separate simulation DB and aggregate telemetry; enforce in code, not convention. | PLAN §7.2: aggregate counts/latencies only; telemetry SDK rejects per-bird/per-account fields. | no | full | Technical telemetry boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION §5: visit invitations use one-time token scoped to one host account. | PLAN §5.5: POST /visits/invite with visitor_email; no friend-of-friend/discovery surfaces in §1.2. | yes | none | Per-invite mechanics survive, but private-relationship/control rationale is absent. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION §5: GET /visits gives invitee email, dates, duration, outstanding invites. | PLAN §5.5: host-authenticated visit log; visit_notifications_enabled default false. | yes | none | Log mechanism survives, but transparency-not-attention-loop rationale is not recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION §5: visitor session has read-only snapshot plus audio access for one host account. | PLAN §5.5: restricted GET /aviary/state for host account; no event writes. | yes | none | Actual snapshot access survives, but no-show-off/marketing rationale is absent. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION §10: 30-60s idle cadence; priority stays polite to avoid announcement-like interruption. | PLAN §10.1: 30-60s cadence, polite aria-live, queued sooner for user events without assertive interruption. | no | full | Slow cadence, screen-reader queue restraint, and anti-announcement pacing survive. |

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | no |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | no |
| F10 | no | no | no |
| F12 | yes | yes | yes |
| F13 | yes | no | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | no | yes |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | no |
| F25 | yes | no | yes |
| F27 | yes | yes | yes |
| F30 | no | yes | yes |
| F31 | no | no | no |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From score JSON intent_recovery.total_possible_weight |
| Reachable gold whys | 47 | S whys always included; F whys included only when captured |
| Excluded unreachable feature whys | 2 | F10 and F19 anchors were not captured |
| Recovered / reachable weight | 91.0 / 146 | Weighted numerator over included whys |
| Whys with reconstruction evidence | 47 | Exact evidence present in frozen reconstruction |
| Whys with PLAN grounding | 47 | Exact plan grounding present |
| rule_without_why cases | 14 | Mechanism survived without the gold rationale |
| plan_only_not_reconstructed cases | 13 | PLAN carried more rationale than B recovered |
| ungrounded_reconstruction cases | 0 | No clear ungrounded why assertions |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 38.0 | 70.4% |
| Affective whys | 92.0 | 53.0 | 57.6% |
| Weight-2 whys | 42.0 | 17.0 | 40.5% |
| Weight-3 whys | 104.0 | 74.0 | 71.2% |
| System-level whys | 28.0 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 118.0 | 68.0 | 57.6% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered better (38.0/54.0) than affective whys (53.0/92.0). The planner preserved server tick, no-LWW, telemetry, and deletion mechanisms especially well; affective exception rationales such as F22, F23, F29, F31, F33, F37, F38, and F39 often became rule-only.
- **Weight-3 vs weight-2.** Weight-3 whys did better (75.0/104.0) than weight-2 whys (16.0/42.0), largely because load-bearing architecture and launch gates were repeatedly explained.
- **System-level vs feature-level.** System-level fidelity (82.1%) is much higher than feature-level fidelity (57.6%). The plan carried philosophy well, but the reconstruction often collapsed local exception rationales into implementation mechanics.
- **Multi-layer recovery patterns.** Primary mechanisms usually survived; secondary examples and downstream consequences were most often dropped. F1/F13 missed the laxer-presence examples, F16 missed the PII/log-leak layer, and F18 missed the overwrite-loss scenario.
- **Subdomain patterns.** Audio, server simulation, privacy telemetry, deletion, and accessibility launch gating were strong. Social visit details and small UI refusals were weaker.
- **Evidence-bound effects.** v06 denied credit where the mechanism was present without the why. The clearest cases are F5, F6, F11, F22, F23, F28, F29, F31, F33, F34, F37, F38, and F39.

The failure shape suggests that a strong implementation plan can preserve most buildable behavior while still losing the product-relationship logic behind small negative constraints.

---

## 4. Recommendations for v2 hardening

- Keep the evidence-bound operator. It cleanly exposed mechanism-only preservation that would be over-credited by a looser pass.
- Add more targeted small negative UI exclusions in future instances. This run missed or compressed no-toast, offer cooldown, top-bar auto-fade, empty-aviary, sync-conflict tone, and privacy-link features.
- Consider adding a category for affective exception rationales. Many missed whys were not architecture failures; they were relationship-framing failures.
- Keep multi-layer whys for high-impact functional surfaces. They discriminated well on presence, no-LWW, synthetic IDs, and drift calibration.
- Preserve the strict cross-cutting system bar, but have scorers document partials explicitly; S2/S4/S6 depended on that judgment here.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The validity audit found no significant contamination signs. The reconstruction follows the plan structure and lacks gold IDs/rubric terminology.
- **Single-run-at-temperature limitation.** This is one run with no variance signal.
- **Borderline capture calls.** I leaned inclusive for glossary/domain terms, per-bird call signature, bird-chosen perch, night state, top-bar contents, palette, visit default-off, and visitor actual-aviary behavior. These did not change the headline from high-planning-quality to low-quality; they move only a few points.
- **System-level cross-cutting.** S2, S4, and S6 are partials because the PLAN visibly preserves them across decisions, but B did not recover every gold layer.
- **Confabulation cases.** I did not find clear ungrounded reconstruction claims that affected recovery credit.
- **Evidence-bound denials.** Several rows had exact feature/mechanism evidence but no why evidence, so v06 scored them as none or partial.
- **Rule-without-why cases.** Fourteen included whys were marked rule_without_why; most were social, relationship, or exclusion rationales.

---

End of report.
