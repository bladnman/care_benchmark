# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: PASS. A targeted search of the frozen reconstruction and plan found no gold IDs or rubric IDs such as `S1`, `F1`, `F40`, or `R-F01`. The reconstruction does not use gold-list identifiers or the external taxonomy.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "already alive when opened" | PLAN.md §1 uses the exact phrase. | Plan-derived. |
| "server-authored simulation" | PLAN.md §16 says the interactions feed a "slow server-authored simulation." | Plan-derived. |
| "same charm" | PLAN.md §16 says accessibility surfaces carry "the same charm." | Plan-derived. |
| "flattened state list" | PLAN.md §14.5 names accessibility becoming a flattened state list. | Plan-derived. |
| "specific" and "non-gamified" | PLAN.md §1 uses both in the implementation goal. | Plan-derived. |
| "no notification badge framework" | PLAN.md §14.6 says no notification badge framework in the aviary top bar. | Plan-derived. |
| "quiet, read-only visits" | PLAN.md §16 uses that phrase. | Plan-derived. |
| "Voice boundaries" | Not exact wording, but PLAN.md §1 and §4.4 define the naturalist vs matter-of-fact split. | Benign synthesis. |

No rubric-side vocabulary such as "intent fidelity", "feature-level fidelity", "multi-layer recovery", "weight-3", "gold why", or "load-bearing" appears in the reconstruction. The phrase `NOT RECOVERABLE FROM PLAN` appears, but that is a reconstruction-workflow marker rather than gold-list content.

## Heading Mirror Check

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, and then `### 1. Product Boundary and V1 Scope` through `### 16. Definition of Done for V1`. The numbered headings mirror PLAN.md's top-level section structure, not GOLD_WHYS.md's S1-S9/F1-F40 headings. This is expected plan derivation, not gold-list mirroring.

## 1:1 Mapping Suspect Check

Verdict: PASS. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve gold order, and does not provide a neat 49-item mapping. Instead, it gives 13 synthesized system bullets and then many plan-section bullets in the same order as PLAN.md. That shape matches the source plan and is not suspicious.

## Plan-Derivation Spot Check

1. Reconstruction: "The server is the only authority while clients render snapshots, synthesize calls, interpolate motion, and submit events."
   PLAN support: §1 has the same invariant, and §2.2 separates server and client responsibilities.

2. Reconstruction: "Visitors can observe a host aviary read-only" and visitor presence "never affects host drift."
   PLAN support: §1 states visitors can observe read-only and never affect drift; §3.7 and §4.3 prevent visitor events.

3. Reconstruction: "Accessibility must ship in V1, not as a later retrofit" and should avoid a "flattened state list."
   PLAN support: §9 opens with that launch requirement; §14.5 names the flattened-state-list risk.

All three articulate sentences are directly grounded in the plan.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold ID leakage, no rubric vocabulary, headings mirror the plan rather than the gold list, and spot checks ground the articulate synthesis in PLAN.md. Minor synthesis phrases such as "voice boundaries" are inferable from the plan and do not look like contamination.
