# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A search of the frozen reconstruction for gold IDs and rubric-style IDs such as F1-F40, S1-S9, and R-Fxx returned no hits. The reconstruction uses product terms and plan-derived headings, not gold identifiers.

## Vocabulary Check

Sampled phrases against PLAN:

| Reconstruction phrase | PLAN support | Concern |
|---|---|---|
| "Affective constraints are acceptance criteria, not aspirations" | PLAN opening uses the same claim. | none |
| "the engine is the value, not the topology" | PLAN §2.1 says this exactly. | none |
| "drift_calibration config" | PLAN §§5.2 and 12 name this config. | none |
| "watching without moving is the product" | PLAN §5.4 says this exactly. | none |
| "distinct, calmer aesthetic" | PLAN §7.5 says reduced motion is this. | none |
| "privacy is an architectural boundary, not a policy footnote" | PLAN §10.2 uses the same architectural-boundary framing. | none |
| "temptations a well-meaning contributor will reach for" | PLAN §11.4 uses this wording. | none |
| "same canonical state" | PLAN §§2.2, 6.1, and 9.1 ground this. | none |

No rubric-only vocabulary such as "multi-layer recovery," "feature-level fidelity," "weight-3," or "intent fidelity" appears in the reconstruction.

## Heading Mirror Check

RECONSTRUCTION headings are: System-level intent, Per-feature whys, Scope, Architecture, Data model, API surface, Simulation engine design, Sync model, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Performance budgets and observability, Engineering structure and guardrails, Drift calibration and test harness, Rollout, Risks/mitigations/deferred specs.

These mirror the PLAN's implementation sections, not GOLD_WHYS section titles. The only generic overlap with the scoring package is "System-level intent" / "Per-feature whys," which is the expected reconstruction format rather than a gold-list mirror.

## 1:1 Mapping Suspect Check

Verdict: no 1:1 gold mapping. The reconstruction does not enumerate S1-S9 or F1-F40 and does not proceed in gold-list order. It organizes by the plan's sections and includes many non-gold implementation items such as Snapshot KV, ETag narration, Postgres partitioning, AudioContext pooling, and rollout order. Some gold-bearing features appear because the plan is comprehensive, but the shape is not a neat gold-target table.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The architecture section says to keep services few because 'the engine is the value, not the topology.'" Plan support: PLAN §2.1 says, "Keep services few; the engine is the value, not the topology." Verdict: plan-derived.

2. Reconstruction sentence: "The plan says email appears only on account.email_encrypted, identifiers are synthetic UUIDs, the analytics warehouse never reads the simulation DB." Plan support: PLAN §3 says email appears only on account.email_encrypted; §10.2 says the simulation DB is never read by analytics. Verdict: plan-derived.

3. Reconstruction sentence: "When prefers-reduced-motion or the opt-in is set: micro-motion -> slow cross-fades between still poses." Plan support: PLAN §7.5 says reduced motion maps micro-motion to slow cross-fades between still poses. Verdict: plan-derived.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no 1:1 gold ordering, and the most articulate claims spot-check directly against PLAN. Minor overlap with scorer concepts comes from the required reconstruction headings, not contamination.
