# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Mechanical search found no gold IDs or rebuild IDs in `RECONSTRUCTION.md`: no `F1`-style, `S1`-style, `R-F##`, or `R-S##` tokens. Because no offending IDs appeared, there was nothing to cross-reference against PLAN. Verdict for this section: clean.

## Vocabulary Check

Sampled load-bearing phrases from the reconstruction and checked them against PLAN:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `long-term observational relationships` | PLAN.md:4 uses the same phrase | plan-derived |
| `Presence-based interaction (idle watching)` | PLAN.md:7 uses the same phrase | plan-derived |
| `Server is the Authority` | PLAN.md:29 uses the same heading/phrase | plan-derived |
| `Client is a Projector` | PLAN.md:30 uses the same heading/phrase | plan-derived |
| `append-only log` | PLAN.md:31 and 57 use the phrase | plan-derived |
| `opt-in, read-only, ambient "Visit"` | PLAN.md:11 uses the same phrase | plan-derived |
| `naturalist-voice observation log` | PLAN.md:12 uses the same phrase | plan-derived |
| `accessibility surfaces ... core features` | PLAN.md:152 says accessibility surfaces are core features | plan-derived |
| `WebAudio-based procedural synthesis` | PLAN.md:114 uses the same phrase | plan-derived |
| `aggressive code-splitting and asset optimization` | PLAN.md:153 uses the same phrase | plan-derived |

Searches for rubric/scorer-side terms (`gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, `intent fidelity`) returned no hits. `NOT RECOVERABLE FROM PLAN` appears repeatedly, but that phrase is part of the reconstructor task format and is used with PLAN-structured feature rows.

## Heading Mirror Check

Reconstruction headings:

| Heading | Mirrors PLAN? | Mirrors gold list? | Assessment |
|---|---|---|---|
| `## System-level intent` | No direct PLAN heading; required output heading | No; generic scorer/reconstructor section | acceptable |
| `## Per-feature whys` | No direct PLAN heading; required output heading | No; generic scorer/reconstructor section | acceptable |
| `### 1. Scope` | Mirrors PLAN section 1 | No | plan-derived |
| `### 2. Architecture` | Mirrors PLAN section 2 | No | plan-derived |
| `### 3. Data Model` | Mirrors PLAN section 3 | No | plan-derived |
| `### 4. API Surface` | Mirrors PLAN section 4 | No | plan-derived |
| `### 5. Simulation Engine Design` | Mirrors PLAN section 5 | No | plan-derived |
| `### 6. Sync Model` | Mirrors PLAN section 6 | No | plan-derived |
| `### 7. Frontend Rendering Pipeline` | Mirrors PLAN section 7 | No | plan-derived |
| `### 8. Performance Budgets and Observability` | Mirrors PLAN section 8 | No | plan-derived |
| `### 9. Rollout` | Mirrors PLAN section 9 | No | plan-derived |
| `### 10. Risks` | Mirrors PLAN section 10 | No | plan-derived |

The reconstruction mirrors PLAN section structure, not the gold-list taxonomy.

## 1:1 Mapping Suspect Check

No 1:1 mapping to S1-S9 or F1-F40 is present. The reconstruction gives 9 system-level principles, but they are expressed in PLAN-derived language and do not align cleanly with the gold IDs or order. The per-feature section follows the PLAN's own sections and subfeatures, including many implementation/API/data-model rows that are not gold why targets. This is not suspicious.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| `Interactions are append-only signals, not direct state edits.` | PLAN.md:31 says clients append events to an append-only log and the server tick consumes it; PLAN.md:30 says clients never write personality or mood directly. | grounded |
| `Naturalist prose as part of the product voice.` | PLAN.md:12 names a naturalist-voice observation log; PLAN.md:121 names naturalist prose narration. | grounded |
| `Accessibility surfaces are core features.` | PLAN.md:13 lists narration/reduced-motion/captions in scope; PLAN.md:152 says narration/captions are core CI/CD features, not add-ons. | grounded |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses PLAN vocabulary, mirrors PLAN headings rather than gold headings, does not leak gold IDs, and does not map neatly to the held-out S/F taxonomy. Some rationales are generic inferences from the PLAN, but they remain grounded in the PLAN rather than in gold-side language.
