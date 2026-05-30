# VALIDITY_AUDIT -- CARE run 001

## ID Leakage

Verdict: no ID leakage found. I searched the frozen reconstruction for gold-style IDs (`S1`-`S9`, `F1`-`F40`, `R-Fxx`) and rubric labels. It does not use gold IDs or a gold-taxonomy scaffold. The word `canonical` appears many times, but it also appears repeatedly in PLAN.md and is ordinary plan vocabulary here, not a gold-list leak.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `one canonical server-authored state` | PLAN line 7 uses the same phrase. | Plan-derived. |
| `protect affective quality` | PLAN line 12 says the architecture narrows scope to protect affective quality. | Plan-derived. |
| `Presence is the dominant drift input` | PLAN line 9 says this exactly. | Plan-derived. |
| `quiet field` | PLAN lines 565 and 569 specify quiet field/no spinner. | Plan-derived. |
| `honesty over faux continuity` | PLAN line 536 uses the phrase. | Plan-derived. |
| `first-class product surface` / `not a degraded fallback` | PLAN lines 596-603, 877-879, and 937 support this. | Plan-derived. |
| `NOT RECOVERABLE FROM PLAN` | This is phase-2A reconstruction protocol language, not gold/rubric content. | Not a contamination sign by itself. |
| `multi-layer`, `weight-3`, `intent fidelity`, `feature-level fidelity` | Not present in RECONSTRUCTION.md. | No rubric-vocabulary concern. |

## Heading Mirror

RECONSTRUCTION headings are: `System-level intent`, `Per-feature whys`, then plan-section headings (`Executive summary and scope`, `System architecture`, `Data model`, `API surface`, `Simulation engine design`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets and observability`, `Rollout plan, risks, and build order`). They mirror the PLAN structure, not GOLD_WHYS section titles or S/F order. No heading mirror issue found.

## 1:1 Mapping Suspect

No near-1:1 mapping to the gold list. The reconstruction has a broad plan-derived inventory with many implementation bullets, including items that are not gold whys, and it does not enumerate S1-S9 or F1-F40. Its order follows PLAN.md sections rather than the gold order.

## Plan-Derivation Spot Check

1. Reconstruction sentence: `Render semantic state locally rather than streaming raw animation.`
   Supporting PLAN passage: PLAN lines 103-113 state that the server sends semantic scene state plus timing anchors, never raw animations, to keep payloads small and support sub-500ms first bird.

2. Reconstruction sentence: `Prefer honesty over fake continuity.`
   Supporting PLAN passage: PLAN lines 536-541 say v1 should prefer honesty over faux continuity and must not allow extended offline simulation that later merges personality changes.

3. Reconstruction sentence: `Make accessibility a first-class product surface, not a degraded fallback.`
   Supporting PLAN passage: PLAN lines 596-603 require a first-class reduced-motion path; lines 877-879 treat narration and reduced-motion as dedicated product surfaces and block launch if they feel stripped.

## Verdict: PASS

The reconstruction reads as plan-derived. I found no gold ID leakage, no rubric vocabulary misuse, no gold-heading mirroring, and no 1:1 gold-list mapping. The strongest sentences are traceable to explicit PLAN passages, and the many `NOT RECOVERABLE FROM PLAN` markers argue against contamination rather than for it.
