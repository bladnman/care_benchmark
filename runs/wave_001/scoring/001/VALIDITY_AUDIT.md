# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found. I searched the frozen reconstruction for gold-style IDs and rubric labels such as `F1`, `F40`, `S1`, `S9`, `R-F01`, `gold`, `rubric`, `intent fidelity`, and `feature-level fidelity`. No gold IDs or external scoring taxonomy appeared in `RECONSTRUCTION.md`. The only searched phrase that appeared was `load-bearing`, and the same phrase appears in the PLAN in the snapshot/delta discussion, so it is not a leakage hit.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "quiet-field loading state persists (no spinner)" | PLAN §7.2 says the quiet-field loading state persists and no spinner is used. | plan-derived |
| "the aviary already in motion" | PLAN §12.5 uses this central-conceit language. | plan-derived |
| "server is the only writer of personality state" | PLAN §2 invariant says this exactly. | plan-derived |
| "two clients independently pulling the same canonical server state" | PLAN §2/§6 describe multi-device sync this way. | plan-derived |
| "Privacy by data-boundary, not just policy" | PLAN §10.5 separates simulation DB from analytics and forbids ETL. | plan-derived paraphrase |
| "same rendering pipeline, the same prose generation" | PLAN §12.4 says accessibility is the same rendering pipeline/prose generation. | plan-derived |
| "load-bearing" | PLAN §6.2 says delta optimization is not load-bearing. | inherited vocabulary, not rubric leakage |
| "affective spine" | PLAN §12.3 says procedural audio is the product's affective spine. | plan-derived |

No rubric-only terms such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, or `intent fidelity` appear in the reconstruction.

## Heading Mirror Check

The reconstruction headings are `System-level intent`, `Per-feature whys`, then plan-shaped headings: `Scope`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine Design`, `Sync Model`, `Frontend Rendering Pipeline`, `Audio Pipeline`, `Accessibility Surfaces`, `Performance Budgets and Observability`, `Rollout Plan`, `Risks`, and `Appendices`. These mirror the PLAN's major sections, not the gold-list headings. They do not mirror `System-level whys`, `Feature-level whys`, or the S/F taxonomy in `GOLD_WHYS.md`.

## 1:1 Mapping Suspect Check

No suspect 1:1 gold mapping. The reconstruction has a broad plan-section inventory with many more than 49 bullets and no S1-S9/F1-F40 ordering. It does not give a neat item for every gold target; for example, several gold-bearing features are marked `NOT RECOVERABLE FROM PLAN` or are absent as distinct why rows. The order follows the plan's sections and implementation surfaces.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Multi-device sync is deliberately described as 'not sync' but 'two clients independently pulling the same canonical server state.'"
   PLAN support: §2 key invariant says multi-device sync is not client-to-client sync, and §6 says devices pull one canonical server state.
   Assessment: supported.

2. Reconstruction sentence: "The deliberate absence of an offline indicator is a 'notice, never announce' decision: an 'offline' banner would announce system state and break felt-aliveness."
   PLAN support: §6.4 says no offline indicator/toast and gives this exact rationale.
   Assessment: supported.

3. Reconstruction sentence: "Bad procedural calls can break the spell and collapse the product's 'affective spine.'"
   PLAN support: §12.3 describes audio uncanniness as breaking the spell and calls audio the affective spine.
   Assessment: supported.

## Verdict

PASS. The frozen reconstruction reads as derived from the PLAN: it uses plan-section headings, repeats or paraphrases plan vocabulary, contains no gold IDs, and does not map neatly onto the gold S/F taxonomy. The one suspicious-sounding term, `load-bearing`, appears in the PLAN itself and is not a contamination signal here.
