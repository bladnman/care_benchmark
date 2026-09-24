# VALIDITY_AUDIT - CARE run 001

## ID Leakage

PASS. Mechanical search of the frozen reconstruction found no gold IDs or external scoring IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. The reconstruction uses plan section numbers such as `§7.3`, not held-out why IDs.

## Vocabulary Check

Sampled phrases all trace to the PLAN rather than scorer-side vocabulary:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "felt aliveness" | PLAN intro names "felt aliveness" as a property built first. | plan-derived |
| "server-authoritative simulation" | PLAN intro and architecture use server-authoritative/canonical state language. | plan-derived |
| "monotonic by construction" | PLAN §5.3 says drift is monotonic by construction. | plan-derived |
| "honest presence" | PLAN intro names honest presence; I3 and §7.1 define it. | plan-derived |
| "no toast/snackbar component" | PLAN I6 and R-V1 describe no toast primitive. | plan-derived |
| "signature syllable" | PLAN §5.9 and §8.3 use signature syllable for recognizability. | plan-derived |
| "quiet field" | PLAN §9.1 uses quiet field for loading/empty states. | plan-derived |
| "no third-party LLM" | PLAN §7.7 gives this privacy rationale. | plan-derived |
| "max_birds_enabled" | PLAN §13.3 names the operational flag. | plan-derived |
| "first-bird" | PLAN I10/§9.1/§11.1 use the first-bird mark and budget. | plan-derived |

No scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` was found in the reconstruction.

## Heading Mirror

The top-level headings are exactly the required reconstruction prompt headings: `## System-level intent` and `## Per-feature whys`. The subsection headings mirror the PLAN's own numbered sections: `0. Invariants and scope`, `2. Architecture`, `3. Data model`, through `16. Definition of done`. They do not mirror GOLD_WHYS section titles or S/F target ordering.

## 1:1 Mapping Suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40 and does not follow the gold list order. It groups feature whys by the plan's sections, includes many non-gold implementation decisions, and marks several plan-local items NOT RECOVERABLE FROM PLAN. This shape is consistent with plan derivation, not a 1:1 gold-target map.

## Plan-Derivation Spot Check

1. Reconstruction: "The server is the source of truth; the client is a renderer." PLAN support: §2.3 says the server decides what is true and the client decides how it looks and sounds; §6.1 says one canonical record, one writer, many readers.

2. Reconstruction: "The notebook writer uses a token bucket to preserve sparsity. Its fact schema has no user-behavior fields." PLAN support: §7.6 specifies the token bucket, spacing caps, and forbids user-behavior fields in notebook facts.

3. Reconstruction: "Accessibility is not a milestone. It's a gate on M2, M5, M6, and M7 exit." PLAN support: §13.1 contains that sentence and lists the gated milestones.

All three articulate plan content directly and have clear supporting passages.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no ID leakage, no scorer/rubric vocabulary, headings mirror the PLAN rather than the gold list, and spot-checked rationale traces cleanly to PLAN passages.
