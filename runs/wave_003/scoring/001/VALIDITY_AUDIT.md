# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: PASS. I searched the frozen reconstruction for gold-style IDs using `S1-S9`, `F1-F40`, and `R-F*` patterns. There were no hits. Running the same pattern over the assigned PLAN also produced no hits, so there is no reconstruction-only gold ID leakage to report.

## Vocabulary Check

| Reconstruction phrase | Plan derivation check | Assessment |
|---|---|---|
| "appears to have been continuing before the viewer arrived" | Exact phrase appears in PLAN.md line 7. | Plan-derived. |
| "the server owns truth; clients render and express it" | Synthesizes PLAN.md lines 52-65 and 280, where the simulation worker owns state and clients render snapshots without vector merge. | Plan-derived synthesis. |
| "Evidence gates matter more than claims" | Not exact, but PLAN.md line 3 says constants need validation gates and section 15 specifies evidence artifacts. It is validation language, not gold/rubric taxonomy. | Minor phrasing watch, not a leakage sign. |
| "Accessibility is the same product" | Exact plan section title at PLAN.md line 368 and echoed by line 370. | Plan-derived. |
| "private relationship data" | Supported by PLAN.md lines 414-418 and 466, which wall off simulation inputs and metrics collection. | Plan-derived. |
| "not a landing-page score" | Exact phrase appears in PLAN.md line 436. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | A reconstruction-format marker, used for plan items where the plan states a rule but no rationale. It is not a gold ID or rubric scoring term. | Expected phase-2A vocabulary. |
| "same scene projection" | Supported by PLAN.md lines 141 and 504. | Plan-derived. |

No rubric-only vocabulary such as "multi-layer recovery", "feature-level fidelity", "weight-3", or "intent fidelity" appears in the reconstruction.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Product contract and release scope`
- `### 2. Explicit decisions where the PRDs leave gaps or conflict`
- `### 3. Architecture and ownership boundaries`
- `### 4. Durable data model`
- `### 5. API contracts and authorization`
- `### 6. Simulation engine and continuity`
- `### 7. Presence, interactions, and session state machines`
- `### 8. Pull synchronization and degraded operation`
- `### 9. Frontend scene and loading pipeline`
- `### 10. Procedural audio and captions`
- `### 11. Accessibility as the same product`
- `### 12. Field notebook and adoption pacing`
- `### 13. Privacy, account lifecycle, and visit details`
- `### 14. Performance budgets and operational observability`
- `### 15. Verification plan and acceptance evidence`
- `### 16. Delivery sequence and rollout`
- `### 17. Risks, mitigations, and final readiness criteria`

These mirror PLAN.md section headings, not GOLD_WHYS.md headings. The gold list is organized as system whys S1-S9, feature whys F1-F40 grouped by PRD file, and the 120-feature list. The reconstruction does not use that structure.

## 1:1 Mapping Suspect Check

Verdict: PASS. The reconstruction does not assign neat items to S1-S9 or F1-F40, does not use those IDs, and does not follow the gold-list order. It instead walks the PLAN's 17 sections and creates many more bullets than the 49 gold whys. Some bullets later bind cleanly to gold whys because the plan is thorough, but the structure is plan-derived rather than gold-derived.

## Plan-Derivation Spot Check

1. Reconstruction line 3 says the aviary "appears to have been continuing before the viewer arrived" and cites mid-action loading, server advance, and first-bird performance. PLAN.md line 7 has the exact continued-before-arrival phrase; lines 296, 308, 441, and 451 support first-paint/first-bird behavior.
2. Reconstruction line 247 says `present` requires visible, focused, and trusted recent pointer/key input, while open tabs and focus alone do not count. PLAN.md lines 236-242 define exactly those view states, exclusions, and bounded segment rules.
3. Reconstruction line 403 says deletion/final erasure/key destruction/journal replay honor the hard-deletion promise across jobs, backups, restores, ticks, exports, mail, and recovery races. PLAN.md lines 424-428 describe the 30-day recovery deadline, full deletion set, key destruction, and backup tombstone replay.

All three articulate sentences are directly supported by the assigned PLAN.

## Verdict

PASS. The reconstruction reads as a dense derivation from the assigned PLAN: no gold IDs leaked, headings mirror the plan rather than the gold list, and spot-checked claims are grounded in plan text. The only mild watch item is validation/evidence language, but the plan itself repeatedly discusses validation gates and evidence artifacts, so this is not enough to flag the run.
