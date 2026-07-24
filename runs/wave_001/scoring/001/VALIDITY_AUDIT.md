# VALIDITY_AUDIT - run 001

## 1. ID Leakage Check

Mechanical search for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, `R-S[0-9]+`, plus scorer-side terms (`gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`) returned no hits in the frozen reconstruction.

Verdict for this section: no gold ID leakage detected.

## 2. Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "Server-owned truth, client-owned presentation" | yes, semantic and quoted via "server computes what is true" / "client computes what it looks and sounds like" | plan-derived |
| "Single canonical state; server-only writes; additive deltas" | yes, exact phrase appears in PLAN §6 | plan-derived |
| "quiet relationship product" | yes, semantic from hard non-goals and quiet/ambient surfaces | plan-derived |
| "Privacy by architectural absence" | yes, semantic from pipeline-level privacy and "We do not compute stats we refuse to surface" | plan-derived |
| "Naturalist ambient voice split from matter-of-fact system voice" | yes, semantic and phrase-level support in PLAN §9 | plan-derived |
| "a separate render path, not a fallback" | yes, exact phrase appears in PLAN §7 | plan-derived |
| "Tamagotchi" / "screensaver" calibration band | yes, exact risk language appears in PLAN §12 | plan-derived |
| "NOT RECOVERABLE FROM PLAN" | not in PLAN | allowed reconstructor marker, not gold-side vocabulary |

No suspicious rubric/gold-side vocabulary appeared.

## 3. Heading Mirror Check

Reconstruction headings are `## System-level intent`, `## Per-feature whys`, then PLAN-shaped subsections: `Scope`, `Architecture`, `API surface`, `Simulation engine design`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets and observability`, `Rollout`, and `Risks and mitigations`.

The two top-level headings are required by the phase-2A prompt. The subsection headings mirror PLAN sections, not gold-list titles or gold ordering.

## 4. 1:1 Mapping Suspect Check

The reconstruction does not create a neat S1-S9 / F1-F40 sequence and does not use gold IDs. Its per-feature pass follows the PLAN's own sections and includes many plan-specific items that are not gold why anchors, plus many explicit `NOT RECOVERABLE FROM PLAN` statements. This is not a 1:1 mapping to the held-out gold list.

## 5. Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan repeats a strict boundary: the server computes 'what is true'; the client computes 'what it looks and sounds like right now.'" | PLAN §2: "The boundary is strict: the server computes what is true; the client computes what it looks and sounds like right now." | directly grounded |
| "The plan wants privacy enforced 'at the pipeline level, not the policy level.'" | PLAN §2: Simulation DB is never read by telemetry; privacy enforced "at the pipeline level, not the policy level." | directly grounded |
| "Reduced motion is called 'a separate render path, not a fallback' and 'not a later fix.'" | PLAN §7: reduced motion is "a separate render path, not a fallback"; PLAN §9: "not a later fix." | directly grounded |

## 6. Verdict

**PASS.** The frozen reconstruction reads as plan-derived. It has no gold ID leakage, no scorer-side vocabulary, no 1:1 gold mapping, and its headings follow the candidate PLAN rather than the held-out gold list. The `NOT RECOVERABLE FROM PLAN` markers also support the validity contract because the reconstructor did not appear to fill missing rationales from outside knowledge.
