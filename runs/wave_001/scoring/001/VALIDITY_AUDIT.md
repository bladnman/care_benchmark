# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found.

A targeted scan for gold IDs and rubric-side identifiers (`F1`-`F40`, `S1`-`S9`, `R-Fxx`, `weight-3`, `feature-level fidelity`, `intent fidelity`, `multi-layer recovery`, `gold`, `rubric`) returned no hits in the frozen reconstruction. The reconstruction uses plan-facing feature labels and prose headings, not gold-list IDs.

## Vocabulary Check

Sampled load-bearing phrases against PLAN.md:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| “quiet field” | PLAN.md §First-frame strategy uses “quiet field” repeatedly. | Plan-derived |
| “not an engagement loop” | PLAN rejects outbound engagement pings, gamification, leaderboards, and notifications. | Plan-derived synthesis |
| “custodial obligation” | PLAN non-goals name “custodial obligation.” | Direct plan phrase |
| “monotonic-toward-expressive” | PLAN scope names “monotonic-toward-expressive drift.” | Direct plan phrase |
| “only writer of personality vectors” | PLAN Simulation Service says this exactly. | Direct plan phrase |
| “events are facts” | PLAN Sync Model says events are facts, not state mutations. | Direct plan phrase |
| “observations, not announcements” | PLAN narration section says user events are written as observations, not announcements. | Direct plan phrase |
| “hard pipeline boundary” | PLAN Telemetry Pipeline section uses this exact phrase. | Direct plan phrase |
| “first-class workstream from day 1” | PLAN Accessibility regressions mitigation uses this exact phrase. | Direct plan phrase |
| “already-running” | PLAN performance budget says first bird under 500ms feels already-running. | Direct plan phrase |

No suspicious rubric vocabulary was found. “Load-bearing” does not appear in RECONSTRUCTION.md.

## Heading Mirror Check

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `**Scope and account surface**`
- `**Aviary simulation and bird engine**`
- `**Interactions**`
- `**Field notebook and product voice**`
- `**Audio pipeline**`
- `**Rendering and frontend behavior**`
- `**Accessibility surfaces**`
- `**Sync, social, and observability**`
- `**Rollout and launch**`

The top two headings match the expected reconstruction format, not the gold list. The remaining headings mirror PLAN.md implementation sections rather than GOLD_WHYS section titles. No exact or near-exact gold-heading mirror was found.

## 1:1 Mapping Suspect Check

No suspicious 1:1 gold mapping. RECONSTRUCTION.md does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold-list order. It instead walks the plan’s scope/architecture/product areas and includes many non-gold plan features such as browser-only client, session tokens, WebSocket fallback, render technology, rollout phases, and instrumentation. That shape is consistent with a plan-derived reconstruction.

## Plan-Derivation Spot Check

1. Reconstruction: “The plan repeatedly rejects engagement mechanics…”
   - PLAN support: non-goals reject gamification, social-network surfaces, notifications, and outbound engagement pings.
   - Assessment: supported.

2. Reconstruction: “The render pipeline ‘never makes simulation decisions’ and ‘only visualizes what the server has already decided.’”
   - PLAN support: §Render pipeline boundary uses those exact phrases.
   - Assessment: directly supported.

3. Reconstruction: “Telemetry is ‘aggregate-only’; the pipeline has a ‘hard pipeline boundary’ and ‘never reads from the state store or event log.’”
   - PLAN support: §Telemetry Pipeline says aggregate-only, hard pipeline boundary, never reads state store/event log.
   - Assessment: directly supported.

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It contains no gold IDs, no rubric vocabulary, no neat S/F mapping, and its most articulate phrases trace to PLAN.md either exactly or as conservative synthesis from adjacent plan passages.
