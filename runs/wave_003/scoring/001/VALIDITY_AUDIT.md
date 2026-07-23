# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS. A mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. The reconstruction does not use gold IDs and does not mirror the scorer's S/F taxonomy.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "last-write-wins failure mode structurally unreachable" | PLAN section 6 uses the same wording. | plan-derived |
| "collapses the relationship into stats" | PLAN Risks: exposing values "collapses the relationship into stats." | plan-derived |
| "quieter, not warier" | PLAN section 5.2 uses this exact rationale. | plan-derived |
| "build artifact, not a review checklist" | PLAN section 2.3 uses the same privacy-boundary phrase. | plan-derived |
| "a parallel render path, not a stripped one" | PLAN section 9 uses the same reduced-motion phrase. | plan-derived |
| "breaks the entire product" | PLAN Risks uses this for audio uncanniness. | plan-derived |
| "the PRD's explicit list" | PLAN hard rule 1 contains this phrase. | mild concern but plan-derived |

No rubric-side phrases such as "gold", "weight-3", "multi-layer recovery", "feature-level fidelity", or "system-level fidelity" appear in the reconstruction. The phrase "the PRD's explicit list" is unusual for a blind reconstruction, but it is copied from the assigned PLAN rather than from scorer-side materials.

## Heading Mirror

The required top headings are `## System-level intent` and `## Per-feature whys`, matching the phase-2A output contract rather than the gold list. The `###` headings under per-feature whys are `Scope`, `Architecture`, `API surface`, `Simulation engine design`, `Sync model`, `Presence accounting`, `Audio pipeline`, `Frontend rendering pipeline`, `Accessibility surfaces`, `Performance budgets & observability`, `Rollout`, `Risks`, and `Appendix A`. These mirror PLAN section headings, not GOLD_WHYS headings. No near-exact gold-list section mirror was found.

## 1:1 Mapping Suspect

Verdict: not suspect. The reconstruction has 10 numbered system principles and plan-section feature bullets, not a neat S1-S9 plus F1-F40 sequence. It marks multiple items `NOT RECOVERABLE FROM PLAN`, which is consistent with plan-derived reconstruction and inconsistent with gold-list access.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Keep aviary state canonical, server-owned, and structurally conflict-free."
   PLAN support: hard rule 3 says the server is the only writer of personality state; section 6 says one canonical record and "there is nothing to merge."

2. Reconstruction sentence: "Preserve personality as felt behavior rather than visible stats."
   PLAN support: hard rule 1 forbids visible vector values; section 2.2 sends only bucketed renderable consequences; Risks says exposing values collapses the relationship into stats.

3. Reconstruction sentence: "Treat accessibility as a first-version surface, not an afterthought."
   PLAN support: section 10 says accessibility surfaces ship with v1, not after, and are in the critical path; scope includes narration, reduced motion, captions, keyboard, and contrast.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold ID leakage, no rubric vocabulary, headings mirror the PLAN rather than held-out materials, and articulate claims can be traced to PLAN passages. The only audit note is the phrase "the PRD's explicit list," but that phrase appears in the PLAN itself and does not indicate gold/rubric contamination.
