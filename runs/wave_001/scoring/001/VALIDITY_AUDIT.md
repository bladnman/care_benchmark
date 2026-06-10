# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: **PASS**.

Mechanical search of the frozen reconstruction found no gold IDs or external target IDs matching `F1`-`F40`, `S1`-`S9`, or `R-F/R-S` patterns. No leakage hits required cross-reference against the plan.

## Vocabulary Check

Sampled load-bearing phrases from `RECONSTRUCTION.md` and checked them against `PLAN.md`:

| Phrase in reconstruction | PLAN support | Assessment |
| --- | --- | --- |
| "load-bearing PRD rule" | PLAN.md final paragraph uses the same phrase. | Plan-derived. |
| "structural enforcement" / "not a reminder" | PLAN.md scope and final paragraph use this exact method. | Plan-derived. |
| "numeric-exposure firewall" | PLAN.md §5.8 names this firewall. | Plan-derived. |
| "privacy line is infrastructure, not policy" | PLAN.md §10.4 uses this exact phrase. | Plan-derived. |
| "one voice" | PLAN.md language/prose sections say shared prose guarantees one voice. | Plan-derived. |
| "quiet field" | PLAN.md boot path names quiet field repeatedly. | Plan-derived. |
| "screensaver guard" | PLAN.md drift calibration and rollout use this phrase. | Plan-derived. |
| "motionless watching is the product" | PLAN.md presence activity window says this. | Plan-derived. |
| "read-only ambient visits" | PLAN.md social scope uses this phrase. | Plan-derived. |

Suspicious scorer-side phrases such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, and `system-level fidelity` do not appear in the reconstruction. The word `PRD` appears, but it also appears throughout the plan and is not by itself a contamination signal.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data model`
- `### API surface`
- `### Simulation engine design`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### Performance budgets and observability`
- `### Rollout`
- `### Risks`
- `### Ambiguities resolved by this plan`
- `### Workstreams and sequencing`

These mirror the phase-2A required output headings plus the plan's own section headings. They do not mirror `GOLD_WHYS.md` section titles or the S/F target ordering.

## 1:1 Mapping Suspect Check

Verdict: **not suspect**. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold list order. It follows the plan order: scope, architecture, data model, API, simulation, sync, frontend, audio, accessibility, performance, rollout, risks, ambiguities, and workstreams. That is exactly the structure the phase-2A instructions asked the reconstructor to use.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
| --- | --- | --- |
| "The boundary rule: bird-changing state lives on the server; frame-only state lives on the client." | PLAN.md §2.2: "if losing it would change the bird, it lives on the server..." | Directly plan-derived. |
| "Privacy minimization is infrastructure, not policy." | PLAN.md §10.4: "The privacy line is infrastructure, not policy." | Directly plan-derived. |
| "Long-term drift must occupy a narrow band between Tamagotchi and screensaver." | PLAN.md §12.1: "Too fast -> Tamagotchi; too slow -> screensaver." | Directly plan-derived. |

## Verdict

**PASS**. I found no significant contamination signs. The reconstruction uses no gold IDs, avoids scorer/rubric vocabulary, follows the plan's own structure rather than the gold list, and its most articulate phrases are directly grounded in PLAN.md. Minor vocabulary that could look benchmark-like, such as "load-bearing," is already present in the plan.
