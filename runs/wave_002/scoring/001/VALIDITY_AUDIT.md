# VALIDITY_AUDIT -- CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found.

Searches for gold/rubric IDs such as `S1`, `F1`, `F40`, and `R-F01` returned no hits in the frozen reconstruction. The PLAN likewise contains no such IDs, so there is no evidence that reconstruction copied gold identifiers or an external scoring taxonomy.

## Vocabulary Check

Sampled reconstruction phrases against PLAN vocabulary and rubric/gold-side vocabulary:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "authoritative source of truth" | PLAN Architecture uses "authoritative source of truth" | plan-derived |
| "semantic state" | PLAN Render Pipeline Boundary uses "semantic state" | plan-derived |
| "slow cadence" | PLAN uses slow cadence for tick and narration | plan-derived |
| "naturalist field-observation product voice" | PLAN says naturalist prose, field notebook, and naturalist narration | plan-derived paraphrase |
| "Aggregate telemetry only" | PLAN Observability uses this phrase | plan-derived |
| "specifically designed aesthetic" | PLAN Reduced-Motion Mode uses this phrase | plan-derived |
| "per-bird audio recognizability" | PLAN Rollout uses this phrase | plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Reconstruction's own uncertainty marker; not gold taxonomy | not suspicious |

No rubric-side phrases such as "multi-layer recovery," "feature-level fidelity," "weight-3," "load-bearing," "gold why," or "intent fidelity" appear in the reconstruction.

## Heading Mirror Check

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data Model`
- `### API Surface`
- `### Simulation Engine Design`
- `### Sync Model`
- `### Frontend Rendering Pipeline`
- `### Audio Pipeline`
- `### Accessibility Surfaces`
- `### Performance Budgets, Observability, Rollout, and Risks`

The two top-level headings match the reconstruction task shape, not the gold list. The section headings mirror PLAN headings closely and do not mirror GOLD_WHYS section titles beyond ordinary domain overlap.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not enumerate S1-S9 or F1-F40 and does not follow the gold order. Its per-feature pass follows the PLAN's sections and bullets: Scope, Architecture, Data Model, API Surface, Simulation Engine Design, Sync Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, and Performance/Rollout/Risks. Several gold targets are absent or marked NOT RECOVERABLE, which is the opposite of a neat gold-list mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "A single canonical server state, not client-owned state." | PLAN Architecture: server is the "authoritative source of truth"; Sync Model: server owns all personality and mood state. | supported |
| "Presence Definition: The strict conjunction of visible tab, focus, and recent input protects Presence Signal Integrity because over-counting presence would make drift artificially accelerate." | PLAN Simulation Engine: strict conjunction of visibility, focus, recent pointer/keypress; Risks: over-counted presence accelerates population-wide drift. | supported |
| "Reduced-motion mode: ... a specifically designed aesthetic ... slow, calming cross-fades between poses." | PLAN Frontend Rendering Pipeline uses "specifically designed aesthetic" and "slow, calming cross-fades between poses." | supported |

## Verdict

PASS. The reconstruction reads as plan-derived: it mirrors PLAN structure, uses PLAN vocabulary, contains no gold IDs or rubric vocabulary, and often marks missing rationale as NOT RECOVERABLE instead of filling it in. Minor paraphrases are supported by nearby PLAN passages and do not rise to contamination concerns.
