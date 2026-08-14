# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as plan-derived: it uses the two required reconstruction headings, follows the candidate PLAN's own section order, includes many plan-specific implementation details, and does not leak gold IDs, rubric vocabulary, or a neat S1-S9/F1-F40 mapping.

## ID Leakage Check

Mechanical search of `RECONSTRUCTION.md` found no gold IDs or external taxonomy tokens such as `F1`, `F40`, `S1`, `S9`, `R-F01`, or `R-S01`. A matching search of `PLAN.md` also found no such tokens. Result: no ID leakage.

The only repeated suspicious-looking phrase found mechanically was `NOT RECOVERABLE FROM PLAN`, which is required by the phase-2A reconstructor prompt and appears throughout the reconstruction when the plan did not articulate a rationale.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "browser-based ambient virtual aviary" | PLAN opening says "browser-based ambient virtual aviary" | direct plan vocabulary |
| "tri-condition presence detector" | PLAN §2.1 uses "tri-condition presence detector" | direct plan vocabulary |
| "Unattended open tabs inflating presence-time" | PLAN §12 uses this exact risk phrase | direct plan vocabulary |
| "single canonical server-simulated aviary state" | PLAN §1.1 uses this phrase | direct plan vocabulary |
| "Hidden Personality Vectors (Server Canonical Only)" | PLAN §3.1 heading uses this phrase | direct plan vocabulary |
| "eliminate repetition" | PLAN §8.1 says synthesized calls eliminate repetition | direct plan vocabulary |
| "Instant Aliveness" | PLAN §7.1 heading uses this phrase | direct plan vocabulary |
| "continued uninterrupted" | PLAN §6 says refocused tabs render as continued uninterrupted | direct plan vocabulary |
| "gently audible in ambient background; never muted" | PLAN §8.2 uses this phrase | direct plan vocabulary |
| "parallel sensory surface" | Not exact in PLAN, but synthesized from accessibility surfaces and fallback captions | benign synthesis, not rubric/gold vocabulary |

No scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, or `intent fidelity` appears in the reconstruction.

## Heading Mirror Check

Reconstruction headings:

| Reconstruction heading | Gold heading comparison | Assessment |
|---|---|---|
| `## System-level intent` | Not a gold section title; required by the phase-2A prompt | allowed |
| `## Per-feature whys` | Not a gold section title; required by the phase-2A prompt | allowed |

The reconstruction uses bullet labels under PLAN section names rather than gold section headings such as `S1 - feels-alive-not-robotic` or `F1 - presence-definition`. No heading mirror issue.

## 1:1 Mapping Suspect Check

The reconstruction does not create one neat item for every S1-S9 or F1-F40 target. Instead, it identifies 11 system-level principles and then walks the PLAN's own sections, including many features that are not gold why anchors and many `NOT RECOVERABLE FROM PLAN` entries. This is the expected plan-derived shape, not a gold-list-shaped reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Presence should matter only when it is real attention." | PLAN specifies a tri-condition presence detector and warns against "Unattended open tabs inflating presence-time." | grounded synthesis |
| "Clients submit discrete events, never absolute state, while devices read the same snapshot cache." | PLAN §6 says clients submit discrete events, "never absolute state," and laptop/mobile read the same snapshot cache. | directly grounded |
| "Audio fallback automatically activating the call captions surface shows captions are part of the core experience." | PLAN §8.2 says AudioContext failure switches to silence and automatically activates captions. | grounded synthesis |

## Final Verdict

PASS. Minor synthesis phrases appear, but they are supported by the PLAN and do not look like rubric or gold-list leakage. The reconstruction's structure, vocabulary, and omissions are consistent with a fresh plan-only reconstruction.
