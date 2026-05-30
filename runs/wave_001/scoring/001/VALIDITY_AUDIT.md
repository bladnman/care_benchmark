# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no gold-ID leakage found. A targeted search for `S1`-style IDs, `F1`-style IDs, `R-F` IDs, `gold`, `rubric`, `weight-3`, `feature-level`, `system-level`, `intent fidelity`, `multi-layer`, and `load-bearing` found no reconstruction hits except the ordinary phrase `lower-fidelity recorded-audio path`, which also appears in PLAN.md:31. No offending gold ID appeared in RECONSTRUCTION.md.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Audit note |
|---|---|---|
| `continuously alive` | PLAN.md:5 says the aviary feels continuously alive. | Plan-derived. |
| `whether or not a client is open` | PLAN.md:12 uses the same phrase. | Plan-derived. |
| `No spinner, no wake-up animation` | PLAN.md:305 says no spinner, no wake-up animation, no fade-from-static. | Plan-derived. |
| `rewards attention without punishing absence` | PLAN.md:5 uses the phrase. | Plan-derived. |
| `Server authorship is a hard boundary` | PLAN.md:59-60 says server is the only writer and clients append events. | Paraphrase, plan-derived. |
| `Privacy is architectural` | PLAN.md:62 says privacy boundary is enforced architecturally. | Plan-derived. |
| `first-class product features` | PLAN.md:17 uses this phrasing for accessibility surfaces. | Plan-derived. |
| `actual product rather than a degraded surrogate` | PLAN.md:548 uses the same phrase. | Plan-derived. |
| `attentive listening, not channel switching` | PLAN.md:342 uses the same phrase. | Plan-derived. |
| `lower-fidelity recorded-audio path` | PLAN.md:31 uses the same phrase. | Not rubric vocabulary; plan-derived. |

No suspicious rubric/gold-side vocabulary was used freely. The reconstruction vocabulary reads like a condensed version of PLAN.md rather than a gold-list mirror.

## Heading Mirror

The two top-level reconstruction headings are `System-level intent` and `Per-feature whys`, which are expected from phase 2A output structure. The `###` headings mirror PLAN.md sections: `Scope and product contract`, `System architecture`, `Domain and data model`, `API surface`, `Simulation engine design`, `Sync and multi-device model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Privacy, security, and compliance shape`, `Performance budgets and observability`, and `Delivery plan, rollout plan, testing strategy, risks, and readiness`. These do not mirror GOLD_WHYS section titles such as `System-level whys`, `Feature-level whys`, or the F1-F40 canonical labels. No heading-level contamination found.

## 1:1 Mapping Suspect

The reconstruction does not enumerate S1-S9 or F1-F40 and does not proceed in gold-list order. It follows the candidate PLAN structure and creates many more plan-derived bullets than gold targets. Some bullets correspond naturally to gold features because the plan itself covers those features, but there is no neat 49-item mapping and no gold IDs. This check passes.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| `The aviary should feel "continuously alive" rather than like an app that starts and stops.` | PLAN.md:5 sets the continuously alive contract; PLAN.md:12 says the simulation runs whether or not a client is open; PLAN.md:303-307 forbids spinner/wake-up/fade-from-static. | Supported. |
| `Privacy is architectural, not just policy.` | PLAN.md:62 says the privacy boundary is enforced architecturally; PLAN.md:392-393 separates simulation data and telemetry stores and excludes per-account/per-bird streams from analytics. | Supported. |
| `Accessibility surfaces are part of the product, not fallback copies.` | PLAN.md:17 names accessibility as first-class product features; PLAN.md:382 says reduced motion is a full experience; PLAN.md:548 requires accessibility paths to deliver the actual product, not a degraded surrogate. | Supported. |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary beyond ordinary plan phrasing, headings mirror the PLAN rather than the gold list, and spot-checked articulate sentences have direct PLAN support. Scores are not flagged for contamination.

