# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS.

Mechanical scan of the frozen reconstruction found no gold-style IDs such as F1-F40, S1-S9, R-F*, or R-S*. The reconstruction uses the PLAN's section numbering and prose labels rather than scorer-side identifiers.

## Vocabulary Check

Verdict: PASS.

Sampled load-bearing phrases from RECONSTRUCTION.md and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Server-owned canonical state" | PLAN.md client/server split says "Server owns all state." | Plan-derived. |
| "conflict prevention (not conflict resolution)" | PLAN.md Sync Model has the heading "Conflict prevention (not conflict resolution)." | Plan-derived. |
| "triple-condition signal" | PLAN.md scope and presence monitor require visibility + focus + activity. | Plan-derived. |
| "No recorded audio files are shipped" | PLAN.md Audio Pipeline uses this exact rule. | Plan-derived. |
| "Privacy boundary by architecture, not just policy" | PLAN.md telemetry pipeline is "architecturally forbidden from reading the Aviary DB." | Plan-derived paraphrase. |
| "Accessibility as a v1 surface" | PLAN.md scope and launch checklist include narration, reduced motion, captions, WCAG, keyboard, and screen-reader testing. | Plan-derived. |
| "not as a popup, not as a streak reward" | PLAN.md Bird-count ramping uses this exact wording. | Plan-derived. |
| "data-control path with a recovery window" | PLAN.md export and deletion endpoints provide export, soft deletion, and recovery. | Plan-derived paraphrase. |

Suspicious scorer-side vocabulary scan found only the ordinary word "recovery" in the account deletion sentence. No rubric phrases such as gold, weight-3, feature-level fidelity, system-level fidelity, multi-layer recovery, or intent fidelity appear.

## Heading Mirror

Verdict: PASS.

RECONSTRUCTION.md headings are:

- ## System-level intent
- ## Per-feature whys
- ### 1. Scope
- ### 2. Architecture
- ### 3. Data Model
- ### 4. API Surface
- ### 5. Simulation Engine Design
- ### 6. Sync Model
- ### 7. Frontend Rendering Pipeline
- ### 8. Audio Pipeline
- ### 9. Accessibility Surfaces
- ### 10. Performance Budgets and Observability
- ### 11. Rollout
- ### 12. Risks

The two top-level headings match the phase-2A reconstruction instructions. The numbered subsection headings mirror PLAN.md, not GOLD_WHYS.md. They do not mirror the gold-list system or feature tables.

## 1:1 Mapping Suspect

Verdict: PASS.

The reconstruction does not enumerate S1-S9 or F1-F40 and does not preserve gold ordering. It follows the PLAN's twelve implementation sections and includes many implementation rows that are not gold whys. Several gold targets are absent, compressed, or marked NOT RECOVERABLE FROM PLAN. That pattern is consistent with blind reconstruction from PLAN.md rather than scorer-list contamination.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The plan prefers 'conflict prevention (not conflict resolution)' through append-only events and clients that never write personality state."
   PLAN support: Sync Model has "Conflict prevention (not conflict resolution)" and lists "Clients never write personality state" and "Event log is append-only."
   Assessment: plan-derived.

2. Reconstruction sentence: "The 'triple-condition signal' exists because over-reporting presence would 'inflate drift across all accounts'."
   PLAN support: Scope lists presence accounting as visibility + focus + activity, and Risk 5 says over-reported presence inflates drift across all accounts.
   Assessment: plan-derived.

3. Reconstruction sentence: "Birds 3-7 become available by aviary age, thresholds are tunable, and the offer appears in account settings with 'no pressure, no expiration countdown'."
   PLAN support: Bird-count ramping lists age thresholds and says the offer is in account settings with no pressure and no expiration countdown.
   Assessment: plan-derived.

## Verdict

PASS.

No significant contamination signatures were found. The reconstruction uses PLAN structure, PLAN vocabulary, and PLAN-derived paraphrases; it does not leak gold IDs, scorer taxonomy, or a neat one-to-one mapping to the held-out gold list. Scores can be treated as valid for this run.
