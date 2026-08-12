# VALIDITY AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. Search of the frozen reconstruction found no gold IDs such as `F1`-`F40`, `S1`-`S9`, `R-F01`, `GOLD`, or `RUBRIC`. The only rubric/gold-adjacent vocabulary hit was `load-bearing`, in the sentence: `Privacy is "load-bearing." The plan names "Privacy architecture (load-bearing)"...`. The same phrase appears in PLAN.md section 2.5, so it is plan-derived rather than leakage.

## Vocabulary Check

Sampled phrases and plan support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `Server owns life; client renders and reports attention` | PLAN product line uses the same phrase. | Plan-derived. |
| `implementation bans, not backlog` | PLAN section 1.2 uses the same phrase. | Plan-derived. |
| `Privacy architecture (load-bearing)` | PLAN section 2.5 heading uses the same phrase. | Plan-derived despite gold-like sound. |
| `No ETL from simdb to analytics` | PLAN section 2.5 states this directly. | Plan-derived. |
| `Accessibility ships on the same day as the visual aviary` | PLAN section 9.1 states this directly. | Plan-derived. |
| `Aliveness is the first frame` | PLAN section 7.1 heading is `Boot sequence (aliveness is the first frame)`. | Plan-derived. |
| `honest presence` | PLAN D15 and drift calibration discussion use this phrasing. | Plan-derived. |
| `Operational health only` | PLAN section 10.5 states this directly. | Plan-derived. |
| `same atom sequence` | PLAN section 9.3 says captions are generated from the same atom sequence. | Plan-derived. |

No suspicious rubric-only vocabulary such as `intent fidelity`, `feature-level fidelity`, `multi-layer recovery`, or `weight-3` appeared in the reconstruction.

## Heading Mirror Check

Reconstruction headings are only:

| Reconstruction heading | Gold-list comparison | Assessment |
|---|---|---|
| `## System-level intent` | Generic required reconstruction scaffold, not a gold why title. | Not suspicious. |
| `## Per-feature whys` | Generic required reconstruction scaffold, not a gold section mirror. | Not suspicious. |

The reconstruction does not mirror the gold headings (`S1 - feels-alive-not-robotic`, file-group headings, or `F1`-`F40`).

## 1:1 Mapping Suspect Check

No 1:1 gold-order mapping was found. The reconstruction has a broad plan-derived list organized by the PLAN's own sections: locked decisions/scope, architecture/data/API, simulation/sync, and frontend/audio/accessibility/performance. It includes many items outside the 49 gold whys and does not enumerate `S1`-`S9` or `F1`-`F40` in order.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `Scope discipline is expressed as schema and protocol discipline.` | PLAN says out-of-scope items `must not leak into schema or protocol design`, and section 1.2 calls non-goals `implementation bans, not backlog`. | Grounded. |
| `Visitor snapshot endpoint never accepts events.` | PLAN D14 says the visitor snapshot endpoint never accepts events; PLAN 4.6 says visitor heartbeat does not write interaction events. | Grounded. |
| `Reduced-motion path ... is a designed second renderer path, not if reduce return.` | PLAN 7.10 says reduced motion is a designed second renderer path and explicitly says not `if (reduce) return`. | Grounded. |

## Verdict: PASS

The frozen reconstruction reads as derived from the candidate PLAN rather than from the gold list or rubric. There are no gold IDs, no gold-heading mirror, no 1:1 target mapping, and the few load-bearing-sounding phrases are traceable to the PLAN itself. Scores are not flagged for contamination.
