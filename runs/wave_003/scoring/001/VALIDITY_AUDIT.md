# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no ID leakage found.

A mechanical scan of the frozen reconstruction for gold-style tokens (F1-F40, S1-S9, R-F*, R-S*) returned no hits. The reconstruction does not use external gold IDs or a scorer taxonomy not present in the PLAN.

## Vocabulary Check

Verdict: clean.

A scan for scorer-side vocabulary (gold, rubric, weight-3, multi-layer recovery, feature-level fidelity, system-level fidelity, intent fidelity, load-bearing) returned no hits. Spot-checked reconstruction phrases are plan-derived:

| Reconstruction phrase | PLAN support |
|---|---|
| "one private, browser-only aviary" | PLAN Section 1 opens with this contract. |
| "only writer of bird personality" | PLAN Section 2 assigns simulation service as only writer. |
| "schema-allowlisted aggregate health data" | PLAN Section 2 describes the operational metrics path. |
| "already at a sampled point in its timeline" | PLAN Section 2 states this for first visible scene. |
| "lowercase, specific, present-tense naturalist observation" | PLAN Section 1 and notebook section use the same voice rule. |
| "visit-notification opt-in default false" | PLAN persistent account record and decision list include this. |
| "never advances canonical simulation time" | PLAN Section 2 states the browser never advances canonical simulation time. |
| "performance, privacy, accessibility, and sync gates" | PLAN rollout section uses these gates. |

## Heading Mirror

The headings are: System-level intent; Per-feature whys; and PLAN-mirroring numbered sections from Product contract through Performance, verification, and rollout. The first two headings are required by the phase-2A prompt. The numbered subsection headings mirror the PLAN structure, not the gold list S/F taxonomy. I do not flag this as contamination.

## 1:1 Mapping Suspect

Verdict: not suspect.

The reconstruction is organized by the PLAN section order and contains many implementation-specific items that do not map neatly to S1-S9 or F1-F40. It does not enumerate gold why IDs, does not use the 49-row scorer order, and includes honest NOT RECOVERABLE FROM PLAN markings for plan items where rationale was not recoverable. This reads like a plan-derived reconstruction rather than a gold-list backfill.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage |
|---|---|
| "Canonical life belongs to the server; the browser renders and interpolates." | PLAN Section 2: simulation service is the only writer; browser owns interpolation/presentation and never advances canonical simulation time. |
| "Privacy and data minimization are product architecture, not a separate compliance layer." | PLAN Sections 2 and 9: aggregate metrics path has no simulation DB access; per-bird stores are not joined to a warehouse/training/recommendation path. |
| "Accessibility ships as part of the core visual/audio experience." | PLAN Section 8: accessibility ships in the same release gate as the visual/audio path and includes narration, captions, reduced motion, keyboard, focus, and contrast. |

All three articulate plan-supported claims. I found no sentence in this spot check that required gold-list access to explain.

## Verdict

PASS.

No significant contamination signatures appeared. The reconstruction uses the PLAN vocabulary and section structure, contains no gold IDs or rubric vocabulary, and its stronger claims are traceable to specific PLAN passages. The few close mirrors are mirrors of the PLAN, not of the hidden gold list.
