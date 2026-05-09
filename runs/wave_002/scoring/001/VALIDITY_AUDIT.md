# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found.

Searches over the frozen reconstruction found no gold IDs or rubric IDs such as `S1`, `F1`, `F40`, `R-F01`, `feature-level`, `intent fidelity`, `multi-layer`, `weight-3`, `gold`, or `load-bearing`. The reconstruction uses ordinary section headings from the PLAN rather than benchmark identifiers.

## Vocabulary Check

| Reconstruction phrase | Appears or is grounded in PLAN? | Assessment |
|---|---|---|
| "observational relationships" | Yes: PLAN scope says "observational relationships with 2-7 birds." | Plan-derived. |
| "Thin Client / Thick Server" | Yes: PLAN architecture uses the exact phrase. | Plan-derived. |
| "simulation continuity and sync correctness" | Yes: PLAN architecture states this rationale verbatim. | Plan-derived. |
| "low-pass filters" | Yes: PLAN simulation engine uses the phrase. | Plan-derived. |
| "monotonic toward 'expressive'" | Yes: PLAN drift process uses the phrase. | Plan-derived. |
| "avoid mechanical synchrony" | Yes: PLAN chorus logic uses the phrase. | Plan-derived. |
| "beep-y or repetitive" | Yes: PLAN audio risk uses the phrase. | Plan-derived. |
| "first-class screen-reader narration" | Yes: PLAN included-v1 accessibility bullet uses the phrase. | Plan-derived. |
| "naturalist prose every 30-60s" | Yes: PLAN accessibility section says this. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | Not a PLAN phrase, but a reconstruction convention for honest non-recovery. | Not contamination. |

No rubric-side vocabulary was used freely in the reconstruction.

## Heading Mirror Check

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, and plan-section mirrors such as `### 1. Scope and v1 Definition`, `### 2. Architecture`, and so on. The two top headings match the expected reconstruction format, not the gold list. The numbered subheadings mirror the PLAN's own section structure, not `GOLD_WHYS.md` sections or F/S ordering.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold order. Instead it walks the candidate PLAN's sections and bullets, including many items with `NOT RECOVERABLE FROM PLAN`. That shape is consistent with plan-derived reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| "The architecture explicitly chooses a 'Thin Client / Thick Server' model 'to ensure simulation continuity and sync correctness.'" | PLAN §2: "The system follows a Thin Client / Thick Server model to ensure simulation continuity and sync correctness." | Supported. |
| "The simulation uses 'Personality Drift' with 'low-pass filters' and drift 'monotonic toward expressive.'" | PLAN §4: "Personality Drift: Apply low-pass filters" and "Drift is monotonic toward 'expressive.'" | Supported. |
| "Chorus logic uses 'staggered start times' to 'avoid mechanical synchrony,' while audio risk mitigation targets calls sounding 'beep-y' or repetitive." | PLAN §§7,10: "Staggered start times... avoid mechanical synchrony" and "Risk of procedural calls sounding 'beep-y' or repetitive." | Supported. |

## Verdict

PASS. The reconstruction reads as plan-derived: it mirrors the PLAN structure, uses PLAN vocabulary, avoids gold IDs and rubric jargon, and often honestly marks missing rationales as `NOT RECOVERABLE FROM PLAN` rather than filling them in from benchmark-side knowledge.
