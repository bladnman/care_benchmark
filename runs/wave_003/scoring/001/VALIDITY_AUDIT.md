# VALIDITY_AUDIT - CARE run 001

## Verdict: PASS

The frozen reconstruction reads as plan-derived. I found no gold ID leakage, no rubric vocabulary, no gold-list heading mirror, and no neat S1-S9/F1-F40 mapping. The reconstruction is organized around the PLAN's own section order and often quotes or paraphrases PLAN language directly. Minor phrasing such as "affect as architecture" is an inference from the plan's "affective constraints" sentence, not a sign of gold-list access.

## ID leakage

- Search target: gold IDs and rubric/gold labels such as S1, F1, F40, R-F01, gold, rubric, weight-3, multi-layer, feature-level fidelity.
- Result: no hits in RECONSTRUCTION.md.
- Cross-check: no offending ID required PLAN lookup because there were no hits.

Verdict for this check: PASS.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "canonical simulation state" | PLAN §1: same phrase | Directly plan-derived. |
| "semantic and parametric descriptors, not pixels" | PLAN §3: same phrase | Directly plan-derived. |
| "already in motion" | PLAN §§1, 8, 19 | Directly plan-derived. |
| "matter-of-fact voice" | PLAN §§1, 4, 7, 10 | Directly plan-derived. |
| "precisely measured input to slow drift" | PLAN §1: same phrase | Directly plan-derived. |
| "Privacy is enforced by data boundaries" | PLAN §1: same phrase | Directly plan-derived. |
| "actual product experience, not a simplified status feed" | PLAN §1: same phrase | Directly plan-derived. |
| "reachable on demand and never pushed" | PLAN §2: same phrase | Directly plan-derived. |
| "affect as architecture" | PLAN §1 says affective constraints are hard technical requirements | Reasonable compression; not rubric vocabulary. |

No suspect rubric-side terms such as multi-layer recovery, feature-level fidelity, weight-3, load-bearing, or intent fidelity appear in the reconstruction.

## Heading Mirror

RECONSTRUCTION.md headings:

- ## System-level intent - matches the phase-2A reconstruction format, not a gold-list section title.
- ## Per-feature whys - matches the phase-2A reconstruction format, not an S/F gold taxonomy.
- ### Product Framing and V1 Scope - mirrors PLAN sections.
- ### Architecture Overview - mirrors PLAN section 3.
- ### Data Model - mirrors PLAN section 4.
- ### API Surface - mirrors PLAN section 5.
- ### Simulation Engine Design - mirrors PLAN section 6.
- ### Sync Model and Correctness - mirrors PLAN section 7.
- ### Frontend Rendering Pipeline - mirrors PLAN section 8.
- ### Audio Pipeline - mirrors PLAN section 9.
- ### Accessibility Plan - mirrors PLAN section 10.
- ### Privacy and Security - mirrors PLAN section 11.
- ### Performance and Observability - mirrors PLAN section 12.
- ### Rollout Plan - mirrors PLAN section 13.
- ### Testing Strategy - mirrors PLAN section 14.
- ### Key Product Decisions, Risks, Guardrails, Workstreams, Definition of Done - condenses PLAN sections 15-19.

No headings mirror GOLD_WHYS.md titles such as specific S/F IDs or canonical feature IDs.

## 1:1 Mapping Suspect Check

The reconstruction does not provide a neat item for every S1-S9 and F1-F40 in gold order. It has ten system-level bullets and a long per-feature pass organized by PLAN section, including many features outside the 40 gold why anchors. The ordering follows the plan (Product Framing, Architecture, Data Model, API, Simulation, etc.), not the gold list.

Verdict for this check: PASS.

## Plan-Derivation Spot Check

1. Reconstruction: "The plan repeats that 'the server owns the aviary's canonical simulation state,' the simulation worker is 'the only writer of personality vectors,' clients 'never mutate personality state'..."
   - PLAN support: §1 and §3 say the server owns canonical state; §3 says the simulation worker is the only writer; §3 says clients never mutate personality state.
   - Assessment: directly plan-derived.

2. Reconstruction: "Presence must use visibility, focus, and recent activity, lean toward 'quiet watching,' and never become 'visit counts, streaks, calendars, or user-facing progress.'"
   - PLAN support: §4 Presence Window has the three-signal rule, says to lean calibration toward quiet watching, and says presence is not exposed as visit counts/streaks/calendars/progress.
   - Assessment: directly plan-derived.

3. Reconstruction: "Reduced-motion rendering... users still see bird identity, current mood, calls, captions, narration, and drift, not a 'broken static fallback.'"
   - PLAN support: §8 and §10 define reduced motion as cross-fades while keeping calls, captions, narration, mood, identity, and drift; §10 says it must not be a broken static fallback.
   - Assessment: directly plan-derived.

## Verdict

PASS. The reconstruction is broad and polished, but its structure, vocabulary, and claims consistently trace back to the plan. There is no evidence of gold-list leakage, rubric vocabulary, or a suspicious S/F target mapping.
