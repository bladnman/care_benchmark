# VALIDITY_AUDIT — CARE run 001

## ID Leakage

Mechanical search of `RECONSTRUCTION.md` for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits. The reconstruction does not use gold IDs or an external ID taxonomy.

## Vocabulary Check

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Preserve the stricter affective invariant" | PLAN §1 uses the exact phrase in the export/hidden-traits decision. | plan-derived |
| "No raw vectors, trait labels or one-to-one numerical trait aliases" | PLAN §2 uses the exact phrase. | plan-derived |
| "two screens cannot double drift" | PLAN §5 uses the exact phrase. | plan-derived |
| "avoid general-purpose toast infrastructure" | PLAN §1 uses the exact phrase. | plan-derived |
| "all accessibility modes at launch" | PLAN §1 uses the exact phrase. | plan-derived |
| "alive accessible aviary" | PLAN §9 uses the phrase in the accessibility testing paragraph. | plan-derived |
| "No launch waiver that postpones a mode" | PLAN §13 uses the exact phrase. | plan-derived |
| "Never compute social rankings or population interaction statistics" | PLAN §1 uses the exact phrase. | plan-derived |

Mechanical search for scorer-side vocabulary (`gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, `intent fidelity`) returned no hits.

## Heading Mirror

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Product boundaries and decisions` through `### 13. Principal risks and responses`

The first two headings are required by the phase-2A reconstruction prompt. The numbered subsection headings mirror the assigned PLAN's own section structure, not the gold-list sections. They do not mirror S1-S9 / F1-F40 ordering or gold section titles.

## 1:1 Mapping Suspect

No 1:1 gold mapping pattern is present. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and organizes per-feature whys under the plan's thirteen implementation sections. It includes many plan-specific implementation items beyond the 49 gold whys, and it marks several items `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction rather than gold-list matching.

## Plan-Derivation Spot Check

1. Reconstruction: "Preserve a hidden affective invariant: birds may be expressive, but the user never sees the machinery." PLAN support: §1 says to "Preserve the stricter affective invariant" by omitting numerical vectors, and §2 says clients receive only observable projections with no raw vectors or trait aliases.
2. Reconstruction: "Attention must be honest but not invasive." PLAN support: §5 requires visibility, focus, and recent pointer/key activity, rejects scroll/timer/audio/network substitutes, merges overlapping devices, and retains no pointer positions or key contents.
3. Reconstruction: "Accessibility is part of the living product, not a late fallback." PLAN support: §1 includes all accessibility modes at launch, §9 says audits alone do not prove an alive accessible aviary, and §13 says there is no launch waiver that postpones a mode.

All three checked sentences are grounded in the assigned PLAN.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it uses the plan's section order and vocabulary, has no gold ID leakage, no scorer-side rubric vocabulary, no near-1:1 mapping to the gold why list, and its most articulate claims are directly traceable to PLAN passages.
