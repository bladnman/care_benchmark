# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching F1/F40, S1/S9, R-F*, or R-S*. The same search over PLAN also returned no such IDs, so there are no reconstruction-only ID hits to cross-reference.

## Vocabulary Check

No rubric-side vocabulary was found by search for gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, system-level fidelity, load-bearing, intent fidelity, CARE, or benchmark.

Plan-derived phrase checks:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "Stable bird UUID, persistent vector and call identity" | PLAN invariant uses the same phrase. | clean |
| "Credit only visible AND window-focused AND recently active owner intervals" | PLAN invariant uses the same phrase. | clean |
| "no ordinary entry sequence, spinner, wake-up, or fade" | PLAN invariant uses the same phrase. | clean |
| "visuals, prose, reduced-motion poses, and captions from the first public release" | PLAN invariant uses the same phrase. | clean |
| "All birds visible in one scene" | PLAN invariant uses the same phrase. | clean |
| "manufacturing unattended hours" | PLAN presence section uses the same phrase. | clean |
| "Keep one chosen behavior per surface" | PLAN risk table uses the same phrase. | clean |

## Heading Mirror

The reconstruction has exactly two headings: `## System-level intent` and `## Per-feature whys`. These mirror the required phase-2A output format, not held-out gold-list headings. There are no S/F-style headings and no close echo of GOLD_WHYS section titles beyond the required structure.

## 1:1 Mapping Suspect

No 1:1 mapping suspect pattern. The reconstruction does not enumerate S1-S9 or F1-F40 and does not use the gold order. Its system section follows PLAN invariants, including extra plan-specific principles such as contradiction handling. Its per-feature section follows PLAN sections and implementation decisions rather than the gold why list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "The plan says browser signals are a cooperative attention approximation and chooses undercounting over manufacturing unattended hours." | PLAN section 7.1 says unreported time is lost and favors a small undercount over manufacturing unattended hours. | grounded |
| "The plan requires shared state and call descriptors to drive visuals, prose, reduced-motion poses, and captions from the first public release." | PLAN invariant states accessible modes carry the same experience from the first public release. | grounded |
| "The plan records tension decisions for raw vectors in export, optional visit notifications, captions/focus outlines, audio autoplay, and settle placement." | PLAN section 2 lists those exact implementation tensions and exceptions. | grounded |

## Verdict

PASS. The frozen reconstruction reads as derived from PLAN structure and vocabulary, with no gold ID leakage, no rubric vocabulary, no heading mirroring of the held-out list, no neat S/F mapping, and spot-checked articulate claims grounded in PLAN passages.
