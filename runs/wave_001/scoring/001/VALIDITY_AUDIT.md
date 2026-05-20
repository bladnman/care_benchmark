# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict for this check: PASS.

I found no gold IDs or rubric-side IDs in `RECONSTRUCTION.md`: no `F1`-`F40`, `S1`-`S9`, `R-F01`, gold, rubric, weight-3, multi-layer, or feature-level fidelity terms. Because there were no hits, no PLAN cross-reference exposed leakage.

## Vocabulary Check

Sampled reconstruction phrases against `PLAN.md`:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "ambient, browser-based experience focused on observation and attention" | Exact phrase appears in PLAN scope. | plan-derived |
| "Monotonic positive drift" | Exact phrase appears in PLAN scope and drift section. | plan-derived |
| "sole writer of the canonical state" | Exact phrase appears in PLAN architecture. | plan-derived |
| "single source of truth" | Exact phrase appears in PLAN sync model. | plan-derived |
| "append-only event log" | Exact phrase appears in PLAN data/sync model. | plan-derived |
| "what state" / "how to render" | Exact render-boundary framing appears in PLAN. | plan-derived |
| "Optional & Quiet" | Exact social heading appears in PLAN. | plan-derived |
| "parallel representation of the same mechanics" | Paraphrases PLAN accessibility sentence "full access to its mechanics through structured alternative representations." | plan-derived |
| "low-power operation and responsive viewports" | Exact phrase appears in PLAN frontend section. | plan-derived |
| "no two synthesized calls sound identical" | Exact phrase appears in PLAN risk mitigation. | plan-derived |

No rubric-side vocabulary appeared in the reconstruction.

## Heading Mirror

The reconstruction headings mirror the PLAN, not `GOLD_WHYS.md`:

| RECONSTRUCTION heading | Closest PLAN heading | Gold-list mirror? |
|---|---|---|
| `## System-level intent` | Phase-2A output structure, not a PLAN heading | Generic scoring structure; not a gold title. |
| `## Per-feature whys` | Phase-2A output structure | Generic scoring structure; not a gold title. |
| `### 1. Scope` | `## 1. Scope` | Mirrors PLAN. |
| `### 2. Architecture` | `## 2. Architecture` | Mirrors PLAN. |
| `### 3. Data Model` | `## 3. Data Model` | Mirrors PLAN. |
| `### 4. API Surface` | `## 4. API Surface` | Mirrors PLAN. |
| `### 5. Simulation Engine Design` | `## 5. Simulation Engine Design` | Mirrors PLAN. |
| `### 6. Sync Model` | `## 6. Sync Model` | Mirrors PLAN. |
| `### 7. Frontend Rendering Pipeline` | `## 7. Frontend Rendering Pipeline` | Mirrors PLAN. |
| `### 8. Audio Pipeline` | `## 8. Audio Pipeline` | Mirrors PLAN. |
| `### 9. Accessibility Surfaces` | `## 9. Accessibility Surfaces` | Mirrors PLAN. |
| `### 10. Performance Budgets and Observability` | `## 10. Performance Budgets and Observability` | Mirrors PLAN. |
| `### 11. Rollout Plan` | `## 11. Rollout Plan` | Mirrors PLAN. |
| `### 12. Risks and Mitigations` | `## 12. Risks and Mitigations` | Mirrors PLAN. |

No heading mirrors gold-list section titles such as `feels-alive-not-robotic`, `notice-never-announce`, or F1-F40 labels.

## 1:1 Mapping Suspect

Verdict for this check: PASS.

The reconstruction does not neatly enumerate S1-S9 and F1-F40 in gold order. It has nine system-level bullets, but they are synthesized from the PLAN and differ from the gold taxonomy. The per-feature section follows the PLAN's sections and bullets, including many items outside the 40 gold why anchors, plus several `NOT RECOVERABLE FROM PLAN` entries. That pattern is consistent with a PLAN-derived reconstruction, not a gold-list mirror.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The architecture insists on one canonical simulation: the server is the `sole writer of the canonical state`, the database is the `single source of truth`, and clients `never send requests like UPDATE birds`." | PLAN architecture says server is the sole writer of canonical state; sync says database is the single source of truth; conflict prevention says clients never send direct mutation requests. | grounded |
| "Conjunct tracking of visible state, focus, and mouse/key activity defines `valid presence` so drift is based on actual attentive presence rather than background time." | PLAN scope defines presence as `visibilityState === visible`, window focus, and mouse/key activity; drift section sums valid presence seconds. | grounded |
| "Procedural synthesis keeps calls unique and species-voiced without `recorded loops`; the plan mitigates `Robotic Procedural Audio` with jitter so no two calls sound identical." | PLAN scope says procedural client-side audio synthesis with no recorded loops; risk mitigation adds jitter so no two synthesized calls sound identical. | grounded |

## Verdict

PASS. The reconstruction reads as plan-derived: it mirrors PLAN headings, uses PLAN vocabulary, contains no gold IDs or rubric vocabulary, and does not map 1:1 onto the gold why list. I found no significant contamination signatures.
