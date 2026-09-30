# VALIDITY_AUDIT — run 001

## ID Leakage

Verdict: no leakage detected. A mechanical search of the frozen reconstruction for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits. The reconstruction does not use gold IDs, rebuild IDs, or an external taxonomy absent from the PLAN.

## Vocabulary Check

No rubric-side terms were found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`. Sampled reconstruction phrases are plan-derived:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "continuity belongs to the birds, not to an open tab" | PLAN.md:5 exact phrase | Plan-derived |
| "the server is the only authority" | PLAN.md:5 exact phrase | Plan-derived |
| "ordinary ambient life and no guilt cues" | PLAN.md:15 exact phrase | Plan-derived |
| "must never credit unattended hours" | PLAN.md:185 exact phrase | Plan-derived |
| "lowercase, present-tense, bird-specific prose" | PLAN.md:203 / :259 close phrasing | Plan-derived |
| "for that user's simulation and nothing else" | PLAN.md:271 exact phrase | Plan-derived |
| "no recorded-audio fallback" | PLAN.md:245 exact phrase | Plan-derived |
| "individually deliberate, revocable, and initially nonexistent" | PLAN.md:273 exact phrase | Plan-derived |

## Heading Mirror

The frozen reconstruction has exactly two headings: `## System-level intent` and `## Per-feature whys`. Those are required by the phase-2A prompt, not copied from the gold list. It does not mirror gold-list section titles such as "System-level whys", "Feature-level whys", or the S/F target names.

## 1:1 Mapping Suspect

No 1:1 gold mapping detected. The reconstruction has 12 system principles and 106 per-feature items in the PLAN's implementation order. It does not enumerate S1-S9 or F1-F40, does not match the 49 gold-why count, and includes many plan-specific items outside the gold why list, such as timezone conflict resolution, SVG-to-Canvas handoff, cache keys, milestones, and rollout gates.

## Plan-Derivation Spot Check

1. Reconstruction: "Watching matters, but unattended tabs do not." PLAN.md:14 says watching without clicking has an effect while unattended/background tabs do not; PLAN.md:181-185 supplies the precise predicate and no-backfill rule.

2. Reconstruction: "Accessibility parity is part of the core aviary, not an add-on." PLAN.md:7 says accessibility and performance are launch requirements, and PLAN.md:17 says audio-off, screen-reader, and reduced-motion users receive the same living aviary.

3. Reconstruction: "Privacy minimization outranks engagement analytics." PLAN.md:271 forbids training/recommendations/population analysis on private bird events, and PLAN.md:296 says dashboards show health/performance rather than engagement or mean drift.

## Verdict

PASS. The reconstruction reads as derived from the PLAN's wording, structure, and priorities. I found no gold-ID leakage, no rubric vocabulary, no held-out heading mirror, and no neat 1:1 mapping to the scorer-side gold list.
