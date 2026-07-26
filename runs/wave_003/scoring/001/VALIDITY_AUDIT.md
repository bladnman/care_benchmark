# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found. A targeted search of the frozen reconstruction found no gold IDs such as `S1`, `S9`, `F1`, `F40`, or `R-F01`, and no loose rubric labels such as `feature-level fidelity`, `multi-layer`, `weight-3`, `intent fidelity`, `gold`, or `rubric`.

The only high-signal vocabulary hit was `load-bearing` in the sentence about export trait exclusion. The same phrase appears in PLAN.md section 1.3 and section 14, so it is plan-derived rather than leakage.

## Vocabulary Check

| Reconstruction phrase | Found in PLAN? | Assessment |
|---|---|---|
| `simulation product with a rendering client` | yes, PLAN section 0 | Directly plan-derived. |
| `world more independent of the viewer` | yes, PLAN section 0 | Directly plan-derived. |
| `architectural boundary, not a policy` | yes, PLAN section 2.1 | Directly plan-derived. |
| `decision on the server, realization on the client` | yes, PLAN section 2.3 | Directly plan-derived. |
| `anti-grind term` | yes, PLAN section 5.3 | Directly plan-derived. |
| `stats panel with extra steps` | yes, PLAN section 4.8 | Directly plan-derived. |
| `felt like a place` | yes, PLAN sections 9.5 and 13.4 | Directly plan-derived. |
| `load-bearing rule` | yes, PLAN sections 1.3 and 14 | Directly plan-derived. |
| `feature-level fidelity`, `weight-3`, `gold`, `rubric` | no in reconstruction | No rubric-side vocabulary present. |

## Heading Mirror Check

The reconstruction headings mirror PLAN.md, not GOLD_WHYS.md. After the required `System-level intent` and `Per-feature whys` headings, the reconstruction uses `0. How to read this plan`, `1. Scope`, `2. Architecture`, through `16. Open items`, matching PLAN.md section order. It does not use gold section headings such as `S1 - feels-alive-not-robotic`, `F1 - presence-definition`, or the F1-F40 canonical order.

Minor note: `System-level intent` and `Per-feature whys` are the expected reconstruction format, not evidence of gold-list mirroring.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not provide a neat S1-S9 then F1-F40 mapping. It is organized around the plan's sections and includes many plan-only implementation details, such as deployment units, data-model choices, WebGL fallback, event validation, rollout milestones, and risk mitigations. Several feature bullets are marked `NOT RECOVERABLE FROM PLAN`, which is inconsistent with a contaminated 1:1 gold reconstruction.

The plan itself is highly structured, so section-by-section reconstruction is expected and not suspicious.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| `Every time a decision is ambiguous, resolve it in the direction that makes the world more independent of the viewer.` | PLAN section 0 contains the same instruction. | Supported. |
| `I9 is an architectural fact, not a policy.` | PLAN section 2.1 says telemetry lives in a different VPC and labels I9 as architectural, not policy. | Supported. |
| `A JSON export can become a stats panel with extra steps.` | PLAN section 4.8 says a JSON file is a stats panel with extra steps and explains the vector-export call. | Supported. |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold ID leakage, no rubric vocabulary, headings mirror PLAN rather than GOLD_WHYS, and articulate sentences trace back to exact PLAN passages. The only notable overlap phrase, `load-bearing`, is present in the plan itself.
