# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS.

Mechanical search found no tokens matching gold/rubric identifiers such as F1-F40, S1-S9, R-Fxx, or R-Sxx in RECONSTRUCTION.md. The same search produced no such identifiers in PLAN.md, so there were no reconstruction-only ID hits to cross-reference.

## Vocabulary Check

Verdict for this check: PASS.

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "small private place" | PLAN definition of complete v1: product remains "a small private place whose birds notice the owner" | Directly plan-derived |
| "canonical life on the server" | PLAN says birds "continue changing on the server" and "There is no client simulation" | Plan-derived paraphrase |
| "qualified, idle attention" | PLAN section 1: "Its main interaction is qualified, idle attention" | Directly plan-derived |
| "absence non-punitive" | PLAN excludes distress/negative drift and says absence is not penalty/guilt | Plan-derived paraphrase |
| "engagement-game framing" | PLAN excludes progress dashboards, streaks, rewards, engagement-driven unlocks | Plan-derived paraphrase |
| "hidden numeric state" | PLAN forbids raw vectors in HTML, snapshots, ARIA, settings, support views, telemetry | Plan-derived paraphrase |
| "first-release obligations" | PLAN says accessibility behavior ships "from the first release" and is not a final-stage retrofit | Directly plan-derived |
| "relationship data separated from operational data" | PLAN section 12.1 has the same separation and technical boundaries | Directly plan-derived |
| "actual authorized bird" | PLAN first-bird evidence requires actual authorized bird, not placeholder | Directly plan-derived |
| "not final-stage retrofits" | PLAN build sequence says accessibility/narration/reduced motion/captions are not final-stage retrofits | Directly plan-derived |

No scorer-side terms appeared in the reconstruction search: no "gold", "rubric", "weight-3", "multi-layer recovery", "feature-level fidelity", "system-level fidelity", "intent fidelity", or "canonical #". The phrase "load-bearing" also did not appear in RECONSTRUCTION.md.

## Heading Mirror Check

Verdict for this check: PASS.

Top-level headings are the required reconstruction-prompt headings: "System-level intent" and "Per-feature whys." The subheadings mirror PLAN.md structure, not GOLD_WHYS.md structure:

| RECONSTRUCTION.md heading | Closest source | Assessment |
|---|---|---|
| Product boundary and binding decisions | PLAN.md section 1 | Plan mirror |
| Decisions where interpretation is needed | PLAN.md section 1.3 | Plan mirror |
| Architecture and ownership | PLAN.md section 2 | Plan mirror |
| Data model and transactional invariants | PLAN.md section 3 | Plan mirror |
| API contract | PLAN.md section 4 | Plan mirror |
| Simulation and temporal continuity | PLAN.md section 5 | Plan mirror |
| Presence and interaction state machines | PLAN.md section 6 | Plan mirror |
| Snapshot consumption, outages, and concurrency | PLAN.md section 7 | Plan mirror |
| Frontend rendering and scene interaction | PLAN.md section 8 | Plan mirror |
| Procedural audio and caption derivation | PLAN.md section 9 | Plan mirror |
| Accessible interaction and narrative | PLAN.md section 10 | Plan mirror |
| Notebook, naming, and gradual adoption | PLAN.md section 11 | Plan mirror |
| Privacy, account lifecycle, and access control | PLAN.md section 12 | Plan mirror |
| Performance, measurement, verification, and rollout | PLAN.md sections 13-17 | Plan mirror/compression |

The headings do not echo GOLD_WHYS.md sections like "System-level whys," "Feature-level whys," or the F1-F40 groupings.

## 1:1 Mapping Suspect Check

Verdict for this check: PASS.

RECONSTRUCTION.md does not create a neat S1-S9 and F1-F40 mapping in gold order. It first lists 13 plan-derived system principles, then many per-feature rationale bullets grouped by PLAN.md sections. It includes implementation constants and operational details that are not gold why anchors, and it marks several exact constants as "NOT RECOVERABLE FROM PLAN." That shape is much closer to the PLAN than to GOLD_WHYS.md.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Make performance part of the product meaning." | PLAN sections 8.1 and 13.1 tie first paint to actual authorized birds, no spinner/placeholder, and the 500 ms first-bird budget as an architecture test. | Supported |
| "Age-based adoption up to seven birds: The plan's rationale is gradual expansion without activity rewards." | PLAN sections 1.1, 1.2, 11.2, and 15.2 specify age-based opportunities, no activity rewards, no manufactured age, and no active-tester reward. | Supported |
| "Relationship data separated from operational data: The plan's rationale is that simulation interaction data advances only that account's aviary and notebook." | PLAN section 12.1 says simulation interaction data is used only for that account's aviary/notebook and enforces this with roles, boundaries, and telemetry allowlists. | Supported |

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it uses the plan's section order, avoids held-out IDs and scorer vocabulary, includes plan-specific implementation details, and its articulate claims have direct support in PLAN.md. I found no significant contamination signatures.
