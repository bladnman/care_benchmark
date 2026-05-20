# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Search of the frozen reconstruction for gold IDs and rubric-side identifiers found no hits for `S1`-`S9`, `F1`-`F40`, `R-Fxx`, `canonical #`, `weight-2`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, or `load-bearing`. No ID leakage was found.

## Vocabulary Check

Sampled phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "slow-timescale relationship" | PLAN line 9 uses the same phrase. | Plan-derived. |
| "idle attention (presence)" | PLAN line 9 uses the phrase. | Plan-derived. |
| "canonical server-side simulation" | PLAN line 23 uses the phrase. | Plan-derived. |
| "Last-Write-Wins" | PLAN line 317 names it. | Plan-derived. |
| "strict PII isolation" | PLAN line 79 uses the phrase. | Plan-derived. |
| "Read-only ambient visits" | PLAN line 29 uses the phrase. | Plan-derived. |
| "integrated directly into the core user experience loop" | PLAN line 403 uses the phrase. | Plan-derived. |
| "retro video game bleeps" | PLAN line 471 uses the phrase. | Plan-derived. |
| "screen-reader queue" | PLAN line 479 uses the phrase. | Plan-derived. |

No rubric-only vocabulary was used freely. The reconstruction does use broad evaluator-like headings, but those headings follow the PLAN, not the gold list.

## Heading Mirror Check

Reconstruction headings are `## System-level intent`, `## Per-feature whys`, and `### 1. Scope` through `### 12. Risks and Mitigations`. The numbered `###` headings mirror the candidate PLAN section headings exactly. They do not mirror GOLD_WHYS section titles such as system-level why IDs or feature-level why groups.

## 1:1 Mapping Suspect Check

The reconstruction does not provide a neat S1-S9/F1-F40 list, does not use gold IDs, and does not follow the gold order. It instead walks the candidate PLAN by section and feature cluster. This is not a 1:1 gold mapping. The broad two-part structure (system intent / per-feature whys) is expected for phase 2A and not suspect by itself.

## Plan-Derivation Spot Check

1. Reconstruction: "V1 focuses strictly on establishing a slow-timescale relationship" and "idle attention (presence) rather than gamified custodial tasks." PLAN line 9 contains the same sentence. Supported.
2. Reconstruction: "multi-device sync via a canonical server-side simulation" and single canonical state. PLAN line 23 and PLAN line 317 support this. Supported.
3. Reconstruction: accessibility targets are "integrated directly into the core user experience loop." PLAN line 403 contains that exact statement. Supported.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as a plan-derived section-by-section synthesis: it mirrors PLAN headings, uses PLAN vocabulary, contains no gold IDs or rubric scoring terms, and does not map neatly onto S1-S9/F1-F40. Scores can be treated as valid for this run.
