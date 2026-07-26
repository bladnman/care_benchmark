# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A search for gold-style IDs in the frozen reconstruction returned no matches. The only ID-like hit in the assigned PLAN was `F0` inside `F0-contour`, which is an audio term and not a gold ID; it did not appear in RECONSTRUCTION.

## Vocabulary Check

Sampled phrases and plan support:

| Reconstruction phrase | In PLAN? | Assessment |
|---|---|---|
| "load-bearing" | yes, PLAN §0 | Not suspicious; inherited from PLAN. |
| "The aviary continues without the viewer" | yes, PLAN §0 | Direct plan vocabulary. |
| "clients send observations of the user" | yes, PLAN §2.2 | Direct plan vocabulary. |
| "monotonic toward expressive" | yes, PLAN §0/§5.2 | Direct plan vocabulary. |
| "structurally incapable" | yes, PLAN §3.6/§10.6 | Direct plan vocabulary. |
| "designed register" | yes, PLAN §7.7 | Direct plan vocabulary. |
| "single most important line of sync code" | yes, PLAN §6.2 | Direct plan vocabulary. |
| "NOT RECOVERABLE FROM PLAN" | allowed reconstructor output | From phase-2A instructions, not gold-side vocabulary. |

No rubric-side vocabulary such as "feature-level fidelity", "system-level fidelity", "multi-layer recovery", "weight-3", or "gold" appears in the reconstruction.

## Heading Mirror Check

RECONSTRUCTION headings mirror the PLAN structure: `0. Reading this plan`, `1. Scope`, `2. Architecture`, through `15. Open questions for the product owner`. They do not mirror GOLD_WHYS section titles or S/F IDs. This is expected because phase-2A was instructed to use the plan's own structure.

## 1:1 Mapping Suspect Check

No 1:1 mapping to S1-S9 or F1-F40 was found. The reconstruction has 10 system-level bullets and many per-plan-section feature bullets in PLAN order, not 9 system rows plus 40 feature rows in gold-list order. Some canonical gold features are merged, omitted, or explicitly marked NOT RECOVERABLE, which argues against gold-list access.

## Plan-Derivation Spot Check

1. RECON: "clients send observations of the user; servers send observations of the aviary." PLAN §2.2 contains the same one-line contract.
2. RECON: "A visitor's attention is structurally incapable of reaching the drift function." PLAN §3.6 says visitor sessions land in a different table that the tick does not read.
3. RECON: "A refusal that lives in a document gets re-argued; a refusal that fails the build gets a conversation with this plan attached." PLAN §12.6 contains the same rationale about executable non-goals.

All three sampled articulate sentences are directly plan-derived.

## Verdict: PASS

The reconstruction reads as plan-derived. It uses the PLAN's headings, order, and vocabulary; contains no unshared gold IDs; avoids scorer-side rubric vocabulary; and includes honest NOT RECOVERABLE markings rather than neat gold-list coverage.
