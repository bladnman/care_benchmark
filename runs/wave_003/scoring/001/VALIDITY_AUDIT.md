# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A targeted search of the frozen reconstruction for gold IDs and rubric IDs (`S1`-`S9`, `F1`-`F40`, `R-Fxx`) returned no hits. The reconstruction uses numbered items, but they follow the PLAN section order rather than the gold taxonomy.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "continuity, procedural variation, and restraint" | PLAN.md:5 uses the same phrase. | Plan-derived. |
| "only writer of canonical bird personality" | PLAN.md:12 and 45-49 use the same authority rule. | Plan-derived. |
| "private simulation inputs" | PLAN.md:170 uses the same phrase. | Plan-derived. |
| "variation is real but debuggable" | PLAN.md:28 uses the same phrase. | Plan-derived. |
| "designed aviary, not a static fallback" | PLAN.md:849 uses the same phrase. | Plan-derived. |
| "notification pressure" | PLAN.md:601 uses the same phrase. | Plan-derived. |
| "cheap to add and expensive to undo" | PLAN.md:21 uses the same phrase. | Plan-derived. |
| "matter-of-fact" | PLAN.md:246, 263, and 706 use it. | Plan-derived. |
| "dominant drift input" | PLAN.md:384 uses the same phrase. | Plan-derived. |
| "first frame should feel already alive" | PLAN.md:559 and 1010 ground the same concept: birds mid-action and aviary already there. | Plan-derived paraphrase. |

No rubric-side phrases such as "intent fidelity", "feature-level fidelity", "multi-layer", "weight-3", or "gold why" appear in the reconstruction.

## Heading Mirror

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Product intent and v1 scope`
- `### 2. Architecture`
- `### 3. Data model`
- `### 4. API surface`
- `### 5. Simulation engine design`
- `### 6. Sync model`
- `### 7. Frontend rendering pipeline`
- `### 8. Audio pipeline`
- `### 9. Accessibility surfaces`
- `### 10. Privacy, security, and data boundaries`
- `### 11. Performance budgets and observability`
- `### 12. Rollout plan, test strategy, risks, and guardrails`

The `###` headings mirror the PLAN's section structure, not GOLD_WHYS section names. The two top-level headings are expected reconstruction-format headings and do not mirror the S/F gold list contents.

## 1:1 Mapping Suspect

No 1:1 gold mapping detected. The reconstruction has 8 system-level bullets and 104 per-feature/plan-derived bullets, not 9 system IDs plus 40 feature IDs. Its order follows the PLAN sections and includes several `NOT RECOVERABLE FROM PLAN` rows for ordinary plan details, which is inconsistent with a leaked gold-list mapping.

## Plan-Derivation Spot Check

1. Reconstruction: "Variation must be deterministic enough to debug." PLAN.md:28 directly supports this with "variation is real but debuggable" for seeded procedural choices.
2. Reconstruction: "Privacy and data minimization are part of the architecture, not an add-on." PLAN.md:70, 170, 242, and 734-740 support the synthetic-ID, private-input, aggregate-telemetry, and no-ML boundaries.
3. Reconstruction: "Accessibility is a first-class product surface." PLAN.md:18, 690, 849, and 963-967 support v1 accessibility, designed reduced-motion/screen-reader surfaces, and avoiding flattened fallback.

All three articulate sentences are grounded in the PLAN.

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no gold-ID leakage, no rubric vocabulary, no gold-heading mirror, and no neat S1-S9/F1-F40 mapping. The strongest evidence is that its headings and item order follow the PLAN, while its vocabulary is mostly direct reuse or close paraphrase of PLAN wording.
