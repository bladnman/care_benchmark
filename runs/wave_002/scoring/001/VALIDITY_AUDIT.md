# VALIDITY AUDIT - CARE run 001

## ID Leakage Check

Verdict: no gold-ID leakage found. I checked for gold IDs and rubric-side markers such as `S1`-`S9`, `F1`-`F40`, `R-F`, `weight-3`, `feature-level fidelity`, `intent fidelity`, and `rubric`. The frozen reconstruction does not use those identifiers. The only nearby term found in PLAN was ordinary prose (`load-bearing libraries`), and it did not appear in the reconstruction as rubric vocabulary.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "enforceable engineering constraints, not tone guidance" | PLAN section 0 uses the same phrase. | Plan-derived. |
| "server decides what, client decides how" | PLAN section 2.3 names this as the central architectural contract. | Plan-derived. |
| "honest signal" | PLAN section 5.3 heading calls presence the honest signal. | Plan-derived. |
| "privacy is implemented as architecture and absence, not policy" | PLAN section 3.4 discusses absences; section 10.5 says architecture, not policy. | Plan-derived synthesis. |
| "designed surfaces, not retrofits" | PLAN section 9 uses this phrase. | Plan-derived. |
| "the product's actual life" | PLAN section 12.2 uses this phrase for warm-path first bird. | Plan-derived. |
| "replayable bug reports" | PLAN section 5.1 uses this wording for deterministic ticks. | Plan-derived. |
| "keeps refusals refused" | PLAN section 3.4 uses this phrase. | Plan-derived. |

No reconstruction phrase sampled here reads like gold-side vocabulary absent from PLAN.

## Heading Mirror Check

Reconstruction headings are `System-level intent`, `Per-feature whys`, and plan-derived implementation headings such as `Scope`, `System architecture`, `Data model`, `Simulation engine`, `Visits`, and `Performance engineering`. These mirror PLAN sections, not GOLD_WHYS section titles. There is no near-exact heading mirror of the S1-S9 or F1-F40 gold labels.

## 1:1 Mapping Suspect Check

The reconstruction does not present a neat S1-S9 or F1-F40 sequence. It lists many more plan-derived implementation bullets in PLAN order, including non-gold implementation details such as object storage, CDN, UUIDv7 primary keys, ETags, service worker cache, perf CI, milestones, and risk controls. That breadth argues against a 1:1 gold-list mapping. Some bullets naturally overlap gold targets because the PLAN itself was comprehensive, but the ordering and naming are plan-derived.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Design principles are meant to bind engineering, not sit above it as tone guidance." PLAN support: section 0 says principles are "enforceable engineering constraints, not tone guidance" and maps each to enforcement.

2. Reconstruction sentence: "Privacy is implemented as architecture and absence, not policy." PLAN support: section 3.4 says missing tables keep refusals refused; section 10.5 labels the telemetry boundary "architecture, not policy"; section 10.4 turns retention into physical privacy.

3. Reconstruction sentence: "Accessibility is product integrity, not compliance afterthought." PLAN support: section 1.1 says accessibility ships with v1, section 9 says these are designed surfaces, and section 15.1 puts narration and keyboard focus in the first alive slice.

All three articulate PLAN content without requiring gold-list access.

## Verdict: PASS

No significant contamination signs were found. The reconstruction reads as a dense plan-derived synthesis: it uses PLAN vocabulary, follows PLAN section order, includes numerous non-gold implementation details, and does not leak gold IDs or rubric terminology.
