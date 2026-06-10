# VALIDITY_AUDIT - CARE run 001

## ID Leakage

No gold-ID leakage found. `RECONSTRUCTION.md` does not use S1-S9, F1-F40, R-Fxx, rubric row labels, or other gold-list IDs. The only suspicious-looking vocabulary hit from the leakage scan was `load-bearing`, and the same term appears in `PLAN.md` in the standing review gate, so it is plan-derived.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Notice-never-announce" | PLAN opening constraints and review gates use the phrase. | Plan-derived. |
| "no announcement-register UI" | PLAN §12 standing review gate. | Plan-derived. |
| "load-bearing" | PLAN §12 says the PRD's load-bearing rules become process. | Plan-derived. |
| "silent-failure class" | PLAN §7 calls presence corruption "the silent-failure class." | Plan-derived. |
| "unreachable, not just discouraged" | PLAN §6 uses that exact framing for last-write-wins. | Plan-derived. |
| "designed register" | PLAN §8 calls reduced motion a designed register. | Plan-derived. |
| "first-bird speed" / 500ms | PLAN §11 time-to-first-bird budget. | Plan-derived. |
| "tone and scope erosion" | PLAN §13 risk about the toast, streak, stats panel. | Plan-derived. |

No rubric-only vocabulary such as "feature-level fidelity," "multi-layer recovery," "weight-3," or "gold why" appears in the reconstruction.

## Heading Mirror

Reconstruction headings are `System-level intent`, `Per-feature whys`, then plan-section mirrors: `Scope`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine Design`, `Sync Model`, `Presence Accounting`, `Frontend Rendering Pipeline`, `Audio Pipeline`, `Accessibility Surfaces`, `Performance Budgets and Observability`, and `Rollout and Risks`.

These headings mirror the PLAN organization, not GOLD_WHYS sections. There is no heading-by-heading mirror of S1-S9 or F1-F40.

## 1:1 Mapping Suspect

No near-1:1 mapping to the gold list was found. The reconstruction has 10 system bullets and many plan-section bullets, not 9 S-items plus 40 F-items in order. Its per-feature section follows PLAN headings and includes many non-gold implementation details such as polling, ETags, sendBeacon, Canvas 2D, AudioContext pooling, and synthetic fleet checks. That shape is consistent with PLAN-derived reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Presence precision is load-bearing." | PLAN §7 defines the three-condition predicate and says the silent failure class gets paid-for test coverage; PLAN §12 uses "load-bearing rules." | Supported. |
| "Sync correctness is designed so last-write-wins on personality is unreachable, not just discouraged." | PLAN §6: "There is no endpoint accepting absolute state - last-write-wins on personality is unreachable, not just discouraged." | Supported. |
| "Reduced motion is a designed register, not an engineering fallback." | PLAN §8: "This is a designed register" and PLAN §10 treats reduced motion as launch-blocking. | Supported. |

## Verdict: PASS

The reconstruction reads as plan-derived. It uses PLAN vocabulary, mirrors PLAN section structure, contains no gold IDs, and does not present a suspicious ordered S/F mapping. Minor vocabulary overlap such as "load-bearing" is explained by the PLAN itself, not by contamination.
