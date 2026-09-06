# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict for this check: PASS. A targeted search of the frozen reconstruction for gold-style IDs (F1-F40, S1-S9, R-F*, R-S*) returned no hits. Because the reconstruction contains no such IDs, there were no ID tokens to cross-check against the PLAN.

## Vocabulary Check

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | Appears in / supported by PLAN? | Assessment |
|---|---|---|
| "appears to have been continuing before it was opened" | PLAN section 1 uses the same phrase. | Plan-derived. |
| "quiet field of sky" without a "spinner" | PLAN section 2.1 uses the same loading-state language. | Plan-derived. |
| "persistent identities and slowly changing personalities underpin the relationship" | PLAN section 1 uses the same phrase. | Plan-derived. |
| "Attention matters without becoming a duty or a score" | PLAN section 1 uses the same phrase. | Plan-derived. |
| "narrow data-portability exception" | PLAN section 1.1 uses this phrase for export vectors. | Plan-derived. |
| "same aviary experience, not a fallback state-list" | Supported by PLAN sections 7.3, 7.4, and 12 even though this exact compression is the reconstructor's phrasing. | Plan-derived synthesis. |
| "launch blockers, not a post-launch phase" | PLAN section 12 uses the same phrase. | Plan-derived. |
| "individually addressed, one-time visit invitations" | PLAN section 1 uses the same phrase. | Plan-derived. |
| "No path renames or replaces the ID" | PLAN section 3 uses the same phrase. | Plan-derived. |
| "canonical" | PLAN repeatedly uses canonical state/record/day language. | Not rubric-side leakage. |

Suspicious scorer-side vocabulary such as "gold", "rubric", "weight-3", "multi-layer recovery", "feature-level fidelity", and "system-level fidelity" did not appear in the reconstruction. The phrase "NOT RECOVERABLE FROM PLAN" appears, but that is explicitly required by the phase-2A reconstruction prompt and is not a gold/rubric leak.

## Heading Mirror

The reconstruction's required top headings are `## System-level intent` and `## Per-feature whys`. Below that, the headings mirror PLAN.md's own structure: `### 1. Product boundary and decisions`, `### 1.1 Explicit interpretations of conflicting or incomplete requirements`, `### 2. Architecture and authority boundaries`, continuing through `### 13. Risks and response`. This is not a mirror of GOLD_WHYS.md, whose scoring structure is system-level whys, feature-level whys grouped by PRD file, complete features list, observed-but-not-planned, and self-check.

Verdict for this check: PASS. The heading mirror is to the PLAN, which was the reconstructor's allowed source.

## 1:1 Mapping Suspect

No 1:1 gold-target mapping was detected. The reconstruction lists 14 system-level principles, not the gold S1-S9 set, and its per-feature section follows the PLAN's implementation headings with many bullets rather than a neat 40-row F1-F40 sequence. It uses no gold IDs and includes plan-only items marked NOT RECOVERABLE where the plan supplied mechanism without rationale.

Verdict for this check: PASS.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The aviary should feel already alive, not launched for the user." Supporting PLAN passages: section 1 says the aviary appears to have been continuing before it was opened; section 2.1 requires existing motion and a quiet no-spinner loading field; section 13 says the first scene should already feel in progress.

2. Reconstruction sentence: "Privacy is structural, not just policy language." Supporting PLAN passages: section 2 gives telemetry no simulation role; section 10 separates simulation storage and operational metrics and forbids analytics/training feeds; section 10 requires schema review and rejected-field tests.

3. Reconstruction sentence: "Accessibility is the same aviary experience, not a fallback state-list." Supporting PLAN passages: section 7.3 requires naturalist narration rather than mechanical lists; section 7.4 calls reduced motion a different rendering of the same aviary; section 12 makes screen-reader, caption, and reduced-motion surfaces launch blockers.

All three articulate sentences are grounded in the plan.

## Verdict

PASS. The frozen reconstruction reads as a plan-derived compression: it mirrors the plan's headings, uses plan vocabulary, includes mandated NOT RECOVERABLE markers for plan-thin rationales, and shows no gold IDs, gold-list ordering, or rubric-side terminology. Scores are not flagged as contamination-suspect.
