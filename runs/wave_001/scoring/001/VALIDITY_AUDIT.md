# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS. I found no significant contamination signs: the reconstruction does not use gold IDs, rubric vocabulary, or a neat S/F mapping, and its structure follows the PLAN section flow rather than GOLD_WHYS.md.

## ID Leakage

- Search for gold IDs and rubric-style IDs in RECONSTRUCTION.md: no hits for S1-S9, F1-F40, R-Fxx, weight-3, multi-layer, feature-level fidelity, intent fidelity, gold, or rubric.
- Cross-reference with PLAN.md: the only search hit in PLAN was the ordinary word fragment "suspend-recovery," not a gold recovery term.

Verdict: PASS.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION.md and checked them against PLAN-derived language:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "only writer of personality state" | PLAN.md repeats exactly in section 2.2, section 6, and section 13 | Plan-derived |
| "affective spine" | PLAN.md section 8 and section 12.3 call audio the affective spine | Plan-derived |
| "quiet field" | PLAN.md section 7.3 and section 12.8 use quiet field for loading | Plan-derived |
| "cumulative-effect argument" | PLAN.md section 1.2 names this argument as final | Plan-derived |
| "relationship deepening" | PLAN.md section 11.2 uses this phrase for new-bird pacing | Plan-derived |
| "tuning knobs, not architectural decisions" | PLAN.md section 14 uses this exact phrase | Plan-derived |
| "graceful silence with captions" | PLAN.md section 8.5 and section 10.2 support this wording | Plan-derived |
| "real watching window" | Derived from PLAN.md presence conjunction and watching-without-moving rationale | Plan-derived paraphrase |

No sampled phrase reads like rubric-only language.

## Heading Mirror

RECONSTRUCTION headings:

| Reconstruction heading | Gold-list mirror? | Note |
|---|---|---|
| ## System-level intent | Generic phase-2A expected heading | Matches expected reconstruction format, not a gold section title |
| ## Per-feature whys | Generic phase-2A expected heading | Matches expected reconstruction format, not a gold section title |
| ### Scope, refusals, and ambiguity calls | No | Mirrors PLAN.md section 1 content |
| ### Architecture, data model, and API surface | No | Mirrors PLAN.md sections 2-4 |
| ### Simulation, sync, rendering, audio, accessibility, and rollout | No | Mirrors PLAN.md later sections |

Verdict: PASS. No near-exact GOLD_WHYS section-title mirroring beyond required generic reconstruction headings.

## 1:1 Mapping Suspect Check

The reconstruction does not create S1-S9 or F1-F40 rows, does not preserve gold order, and includes many non-gold plan items such as magic-link sign-in, service boundaries, render hot/cold paths, WebAudio fallback, browser support, and calibration knobs. Its per-feature sequence follows the plan own scope/architecture/simulation organization.

Verdict: PASS.

## Plan-Derivation Spot Check

1. Reconstruction: "The server is the only writer of personality state."
   PLAN support: section 2.2 says exactly, "The server is the only writer of personality state."
   Result: plan-derived.

2. Reconstruction: "The plan treats the cumulative-effect argument as final."
   PLAN support: section 1.2 says, "The cumulative-effect argument in the PRD is final; the rule is absolute."
   Result: plan-derived.

3. Reconstruction: "Reduced motion is a calmer, slower aviary ... not a broken-looking one."
   PLAN support: section 7.4 says reduced-motion users get "a calmer, slower aviary, not a broken-looking one."
   Result: plan-derived.

## Verdict Rationale

PASS. The reconstruction is highly plan-derived and sometimes borrows exact plan phrasing. It is compressive and occasionally over-paraphrases rationale, but I found no gold-ID leakage, no rubric vocabulary, no gold-heading mirroring, and no neat one-to-one gold-list mapping.
