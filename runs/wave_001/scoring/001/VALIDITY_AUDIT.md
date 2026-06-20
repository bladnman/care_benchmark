# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage

PASS. A mechanical search of the frozen reconstruction found no gold IDs or rubric IDs such as `F1`, `F40`, `S1`, `S9`, `R-F01`, or `R-S01`. No surrounding-sentence leakage hits were present. Because there were no hits, no PLAN cross-reference showed an ID present in reconstruction but absent from PLAN.

## Vocabulary Check

Sampled phrases from `RECONSTRUCTION.md` and PLAN grounding:

| Reconstruction phrase | PLAN support | Verdict |
|---|---|---|
| "server-authoritative simulation" | PLAN opening assumption uses the same phrase. | clean |
| "single canonical aviary per account" | PLAN opening assumption uses the same phrase. | clean |
| "Neglect -> delta = 0" / frozen not decay | PLAN §5.3 says neglect produces zero drift and traits freeze, not decay. | clean |
| "quiet field" | PLAN §§7.1 and 7.7 use quiet field shell/state. | clean |
| "no spinner" | PLAN §§7.1, 7.7, and acceptance criteria forbid spinners. | clean |
| "watching without moving" | PLAN §1.3 uses this phrase for the presence activity window. | clean |
| "read-only ambient visitor view" | PLAN §1.1 uses this phrase in scope. | clean |
| "Accessibility as afterthought" | PLAN §12 risk row uses this exact risk label. | clean |
| "first bird within 500ms" | PLAN §§1.1, 7.1, 10.1, and 14 specify this target. | clean |

No rubric-side phrases such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing` appeared in the reconstruction.

## Heading Mirror

Reconstruction headings:

| Reconstruction heading | Closest gold heading | Assessment |
|---|---|---|
| `## System-level intent` | Gold §1 `System-level whys` | Generic required output heading; acceptable. |
| `## Per-feature whys` | Gold §2 `Feature-level whys` | Generic required output heading; acceptable. |
| `### 1. Scope` | No gold section mirror; mirrors PLAN §1. | clean |
| `### 2. Architecture` | No gold section mirror; mirrors PLAN §2. | clean |
| `### 3. Data Model` | No gold section mirror; mirrors PLAN §3. | clean |
| `### 5. Simulation Engine Design` | No gold section mirror; mirrors PLAN §5. | clean |
| `### 6. Sync Model` | No gold section mirror; mirrors PLAN §6. | clean |
| `### 7. Frontend Rendering Pipeline` | No gold section mirror; mirrors PLAN §7. | clean |
| `### 8. Audio Pipeline` | No gold section mirror; mirrors PLAN §8. | clean |
| `### 9. Accessibility Surfaces` | No gold section mirror; mirrors PLAN §9. | clean |
| `### 10. Performance Budgets and Observability` | No gold section mirror; mirrors PLAN §10. | clean |
| `### 11-14. Rollout, Risks, Workstreams, Acceptance` | No gold section mirror; mirrors PLAN grouping. | clean |

The two required top-level headings are expected by the reconstructor prompt. The remaining headings mirror the PLAN's structure, not `GOLD_WHYS.md`.

## 1:1 Mapping Suspect

PASS. The reconstruction does not provide a neat S1-S9 or F1-F40 mapping and does not follow the gold ordering. Its per-feature section follows the PLAN headings and skips PLAN section 4 in the same way the reconstructor chose to group material. It also marks multiple items `NOT RECOVERABLE FROM PLAN`, which is consistent with a plan-derived reconstruction rather than a gold-list fill-in.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Aliveness comes from gradual, variable, procedural behavior." | PLAN §§5.3-5.5 describe weeks-not-sessions drift, mood persistence, procedural call grammar, and greetings that never repeat exact parameters. | grounded |
| "Accessibility is part of the real aviary experience, not an afterthought." | PLAN §12 risk row says accessibility as afterthought would exclude users from real experience and mitigates by shipping narration/reduced-motion with core. | grounded |
| "Social features must stay narrow, opt-in, and read-only." | PLAN §§1.1, 1.2, 4.6, 6.4, and 12 define opt-in visits, read-only visitor snapshots, revocation, no discovery endpoints, and social-scope-creep mitigation. | grounded |

## Verdict: PASS

The reconstruction reads as plan-derived. It uses the PLAN's structure and vocabulary, contains no gold IDs or rubric vocabulary, and its most articulate claims are grounded in identifiable PLAN passages. The only gold-like headings are the two headings mandated by the reconstructor prompt, so the run's scores do not appear contaminated by held-out scoring material.
