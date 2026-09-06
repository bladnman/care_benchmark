# VALIDITY_AUDIT - CARE run 001

## ID Leakage

No gold IDs or rebuild IDs were found in the frozen reconstruction. A targeted search for tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits. Because no IDs were present, there is no PLAN cross-reference leakage hit.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION.md and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "already in motion when opened" | PLAN.md:9 | Direct PLAN phrase. |
| "moods and ambient behavior continue on the server while nobody watches" | PLAN.md:9 | Direct PLAN phrase. |
| "there is only one history" | PLAN.md:250 | Direct PLAN phrase. |
| "No numbers to optimize" | PLAN.md:523 | Direct PLAN phrase. |
| "specific, lowercase, and observational" | PLAN.md:523 | Direct PLAN phrase. |
| "private state is never used for population-level drift calibration" | PLAN.md:63 | Direct PLAN phrase. |
| "ship with the first usable slice" | PLAN.md:11 | Direct PLAN phrase. |
| "deliberately invited read-only visits" | PLAN.md:11 | Direct PLAN phrase. |
| "Alive becomes canned" | PLAN.md:505 | Mirrors a PLAN risk-register label. |
| "planned is not evidence" | PLAN.md:473 | Direct PLAN phrase. |

No scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, or `intent fidelity` appears in RECONSTRUCTION.md.

## Heading Mirror

The only Markdown `##` headings in RECONSTRUCTION.md are `System-level intent` and `Per-feature whys`, matching the required reconstructor output shape rather than the gold list. The bold subsection headings under per-feature whys mirror PLAN section headings (for example, "Architecture and ownership", "Persistent data model", "Read-only visits", "Performance budgets and operational instrumentation"), not GOLD_WHYS.md section titles or S/F IDs.

## 1:1 Mapping Suspect

No neat S1-S9 or F1-F40 mapping is present. The reconstruction lists 12 system-level principles, then many plan-ordered bullets grouped by PLAN sections. It includes plan-specific implementation choices and several `NOT RECOVERABLE FROM PLAN` items. This is not a one-to-one gold-target reconstruction.

## Plan-Derivation Spot Check

1. RECONSTRUCTION.md:9 says the relationship is "slow, measured, and nonpunitive." PLAN.md:9 describes slow measured attention; PLAN.md:145-157 defines slow positive-evidence drift; PLAN.md:161 says absence never lowers traits; PLAN.md:519 calls the aviary nonpunitive. Supported.

2. RECONSTRUCTION.md:15 frames privacy and data minimization as product boundaries. PLAN.md:63 bars private state from population calibration/training/analysis; PLAN.md:414-420 defines aggregate-only metrics and rejects forbidden labels; PLAN.md:512 makes analytics-as-relationship-dataset a named risk. Supported.

3. RECONSTRUCTION.md:17 says accessibility is a complete presentation, not a patch. PLAN.md:11 says narration, captions, reduced motion, keyboard access, and instrumentation ship with the first usable slice; PLAN.md:378-390 defines the complete accessibility surface; PLAN.md:523 says these are complete at launch. Supported.

## Verdict: PASS

The reconstruction reads as plan-derived. It uses the plan's own structure and vocabulary, has no gold-ID leakage or scorer-side terminology, and its most articulate claims trace back to explicit PLAN passages. Minor heading similarity is expected because the reconstructor was required to use two generic output headings and otherwise followed the PLAN's section order.
