# VALIDITY_AUDIT - CARE run 001

## ID leakage

PASS. Mechanical search found no `S#`, `F#`, `R-S#`, or `R-F#` gold identifiers in the frozen reconstruction. The reconstruction uses plan section numbers and plan vocabulary, not gold-list IDs.

## Vocabulary check

Sampled phrases all appear plan-derived:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `server-authoritative life, not client-authored state` | PLAN repeatedly says server owns canonical state and client never simulates. | Plan-derived synthesis. |
| `Structural prevention over policy or UI omission` | PLAN forbids disabled gamification paths, no spinner path, no recorded-audio fallback, token-scope visitor writes. | Plan-derived synthesis. |
| `visible and tunable rather than buried` | PLAN introduction defines `[CALIBRATE]` and `[DEFAULT]` markers. | Directly plan-derived. |
| `byte-identical twice` | PLAN §4.6/§9.1 uses the same byte-identical wording for calls. | Direct phrase support. |
| `stripped-fallback failure mode` | PLAN §8.6 uses this phrase for reduced motion. | Direct phrase support. |
| `visit-frequency aggregates` | PLAN §12.3 excludes account-level visit-frequency aggregates. | Direct phrase support. |
| `load-bearing` | Appears in PLAN §4.3 and §13.4, so not rubric-only leakage here. | Not suspect. |

No rubric-only vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `intent fidelity` appears in RECONSTRUCTION.md.

## Heading mirror

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then `### 0. Reading this plan` through `### 14. Risks`. These mirror the PLAN section headings, not GOLD_WHYS headings. They do not mirror S1-S9/F1-F40 titles or gold-list grouping names.

## 1:1 mapping suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40 and does not proceed in gold order. Its per-feature section follows the PLAN's 0-14 section order, with bullets matching plan implementation sections. This is the expected shape for a plan-derived reconstruction.

## Plan-derivation spot check

1. Reconstruction: `The tick must run for accounts with no connected client and no new events.`
   PLAN support: §4.1 says the tick queries accounts due for ambient-only advancement and must run without connected clients or new events.

2. Reconstruction: `Pattern functions excluding visit-frequency aggregates ... make a "you've been here every day this week" entry structurally unreachable.`
   PLAN support: §12.3 says pattern inputs are bird/world state, not account-level visit-frequency aggregates, making that entry structurally unreachable.

3. Reconstruction: `No recorded-audio fallback path ... remove the "quick fallback" under deadline pressure.`
   PLAN support: §9.4 says no recorded-audio fallback path exists and no recorded-audio asset pipeline ships.

All three articulate plan content rather than importing gold/rubric language.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold ID leakage, no gold-heading mirror, no 1:1 gold mapping, and the sampled vocabulary is supported by PLAN.md. The only notable overlap with scorer-side language is `load-bearing`, but that word appears in the PLAN itself, so it is not a contamination signature for this slot.
