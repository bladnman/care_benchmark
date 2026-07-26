# VALIDITY_AUDIT - run 001

## Gold ID Leakage Check

Mechanical search found no reconstruction tokens matching gold IDs such as F1-F40, S1-S9, R-Fxx, or R-Sxx. The reconstruction uses plan-native labels such as INV-01 and A-04, which also appear in the PLAN and are not gold-side identifiers.

Verdict for this check: PASS.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION and plan support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "single most load-bearing boundary" | PLAN line 162 uses the same phrase for the authority line. | plan-derived |
| "invariant with no gate is an intention" | PLAN line 1324 states this exactly. | plan-derived |
| "the server owns state; the client owns presentation" | PLAN line 164 states this exactly. | plan-derived |
| "watching without moving is the product" | PLAN line 600 explains this presence-window choice. | plan-derived |
| "privacy is architectural" | PLAN section 12.7 builds physical separation and no-route controls. | plan-derived |
| "not one that looks broken" | PLAN section 8.9 says reduced motion should be calmer, not broken. | plan-derived |
| "goldens" | PLAN line 583 and testing sections use calibration/golden language. | plan-derived |

No scorer-side terms such as "multi-layer recovery", "feature-level fidelity", "system-level fidelity", "weight-3", or "intent fidelity" appear in RECONSTRUCTION. The word "load-bearing" is not suspicious here because it is present in the PLAN.

## Heading Mirror Check

RECONSTRUCTION headings are:

- System-level intent
- Per-feature whys
- 0. How to use this document
- 1. Scope
- 2. Product invariants
- 3. Architecture
- 4. Data model
- 5. API surface
- 6. Simulation engine
- 7. Sync model
- 8. Frontend rendering pipeline
- 9. Audio pipeline
- 10. Voice, copy, and generated prose
- 11. Accessibility
- 12. Accounts, auth, and privacy
- 13. Visits
- 14. Performance budgets and observability
- 15. Testing strategy
- 16. Rollout
- 17. Risks
- 18. Assumptions and judgment calls
- 19. Open items owned outside this plan

These mirror the PLAN's structure, not GOLD_WHYS.md. GOLD_WHYS.md uses S1-S9 and F1-F40 headings grouped by PRD file plus the complete feature table. The only overlap is the required phase-2A output headings, "System-level intent" and "Per-feature whys".

## 1:1 Mapping Suspect Check

No neat 1:1 gold mapping is present. The reconstruction contains 12 system-level bullets and then a plan-section walk from section 0 through 19. It does not list S1-S9, F1-F40, gold titles, weights, or PRD-file-grouped F entries. The order follows PLAN.md rather than the gold list.

## Plan-Derivation Spot Check

1. RECONSTRUCTION: "The server owns state; the client owns presentation."
   PLAN support: line 164 states exactly that, and lines 166-168 define the authority boundary.

2. RECONSTRUCTION: "watching birds without moving is the actual product."
   PLAN support: line 600 says watching without moving is the actual product and explains why the activity window is long.

3. RECONSTRUCTION: "privacy is architectural, not a promise layered on later."
   PLAN support: section 12.7, especially lines 1224-1230, describes no analytics credentials, no network route, allowlisted metrics, and synthetic cohorts.

All three articulate sentences have direct PLAN grounding.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it avoids gold IDs and scorer vocabulary, follows the plan's section order, and its strongest phrases are directly supported by the PLAN. Scores can be treated as valid for this run.
