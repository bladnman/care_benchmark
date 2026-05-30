# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. The reconstruction does not use gold IDs such as `S1`-`S9`, `F1`-`F40`, `R-F01`, or rubric-specific IDs. The only regex hit resembling rubric vocabulary was the ordinary phrase `system-level motion preferences`, which is not a gold ID and is directly grounded in the PLAN reduced-motion section.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| `continuously alive, specific, and private across weeks of use` | PLAN §1 uses the exact phrase. | pass |
| `presence must matter without becoming obligation` | PLAN §1 uses the exact phrase. | pass |
| `server must own canonical bird identity and drift` | PLAN §1 uses the exact phrase. | pass |
| `accessibility and privacy must ship as first-class parts` | PLAN §1 uses the exact phrase. | pass |
| `product risk is in simulation feel, not infrastructure novelty` | PLAN §4 uses the exact phrase. | pass |
| `specific, sparse, safe, and consistent` | PLAN §3 uses the exact phrase for rule-based prose generation. | pass |
| `coherence across devices and in visitor mode` | PLAN §4 uses the exact phrase. | pass |
| `No naturalist copy on these surfaces` | PLAN §9 uses the exact phrase. | pass |
| `relationship as a KPI artifact` | PLAN §17 uses the exact phrase. | pass |
| `first-bird presence over UI richness` | PLAN §19 uses the exact phrase. | pass |

No suspicious rubric-side terms such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `gold why`, or `intent fidelity` appear in the frozen reconstruction.

## Heading Mirror Check

The reconstruction has the required top-level headings `System-level intent` and `Per-feature whys`, then mirrors the PLAN's numbered section headings: `Intent and planning stance`, `Scope`, `Decision calls and assumptions`, through `Execution summary`. These headings do not mirror `GOLD_WHYS.md` sections such as `System-level whys`, `Feature-level whys`, or individual S/F titles. This is plan-derivation evidence rather than contamination evidence.

## 1:1 Mapping Suspect Check

No suspicious 1:1 mapping to the gold list was found. The system section has 11 bullets, not 9 S-whys, and the per-feature section follows the PLAN's 19 sections rather than the 40 F-whys or their gold order. The reconstruction includes many plan-specific implementation rows that have no gold counterpart, and it marks several plan details as `NOT RECOVERABLE FROM PLAN`, which is inconsistent with gold-list leakage.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| `The product is intentionally not about feature breadth.` | PLAN §1: `The product goal is not feature breadth`. | supported |
| `The client must never invent canonical bird behavior.` | PLAN §4 Render boundary uses the exact sentence. | supported |
| `Day-one instrumentation is sufficient to keep the product healthy without measuring the user relationship as a KPI artifact.` | PLAN §17 uses the same KPI-artifact sentence. | supported |

## Verdict: PASS

The reconstruction reads as a faithful PLAN-derived artifact. It mirrors PLAN structure, quotes PLAN vocabulary heavily, contains no gold IDs or rubric-only scoring terms, and does not map neatly onto the 9 system and 40 feature gold targets. Scores are not suspect on contamination grounds.
