# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

Mechanical search of the frozen reconstruction for gold IDs or external benchmark IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+` returned no hits. The reconstruction does not use gold why IDs, rubric IDs, or the external target taxonomy.

## Vocabulary Check

Sampled load-bearing phrases from the reconstruction and checked them against PLAN vocabulary:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "The server owns the bird; the client renders it" | PLAN intro uses the same sentence | plan-derived |
| "Architectural absence is a defense" | PLAN 1.2: "architectural absence is the defense" | plan-derived |
| "lunch-phone-overwrites-morning-laptop failure" | PLAN 5.2 names the same failure | plan-derived |
| "graded affective contract" | PLAN 6.2: "graded affective contract" | plan-derived |
| "audible signature of dead software" | PLAN 7.1 uses the same phrase | plan-derived |
| "designed for charm, not parity-by-checklist" | PLAN 8 uses the same phrase | plan-derived |
| "privacy is architectural, not policy" | PLAN 9.1 uses this framing | plan-derived |
| "small social system, not a row of NPCs" | PLAN 4.8 uses this exact contrast | plan-derived |

Suspicious scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, and `intent fidelity` were not found in the reconstruction.

## Heading Mirror

Reconstruction headings:

| Heading | Comparison to gold/rubric structure | Assessment |
|---|---|---|
| `## System-level intent` | Required by phase-2A prompt, not a gold-list title beyond the required scaffold | allowed |
| `## Per-feature whys` | Required by phase-2A prompt, not a 1:1 gold mirror | allowed |
| `### Scope` | Mirrors PLAN section 1 | plan-derived |
| `### Architecture` | Mirrors PLAN section 2 | plan-derived |
| `### Data model` | Mirrors PLAN section 3 | plan-derived |
| `### Simulation engine` | Mirrors PLAN section 4 | plan-derived |
| `### Sync and error surfaces` | Mirrors PLAN section 5 plus plan copy section | plan-derived |
| `### Frontend rendering pipeline` | Mirrors PLAN section 6 | plan-derived |
| `### Audio pipeline` | Mirrors PLAN section 7 | plan-derived |
| `### Accessibility surfaces` | Mirrors PLAN section 8 | plan-derived |
| `### Social` | Mirrors PLAN social subsection | plan-derived |
| `### Privacy, performance budgets, and observability` | Mirrors PLAN section 9 | plan-derived |
| `### Rollout, calibration, and build sequence` | Mirrors PLAN sections 10-12 | plan-derived |

No heading sequence mirrors S1-S9 or F1-F40. The order follows the PLAN, not GOLD_WHYS.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not produce neat S1-S9 or F1-F40 entries. It uses 11 system-level bullets and then plan-section groupings with many implementation items, including features that are not gold-why anchors. The order tracks PLAN.md sections rather than the gold list. Several gold targets have no corresponding recovered item, which is itself evidence against a leaked 1:1 answer key.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The server owns the bird; the client renders it." | PLAN intro: "the server owns the bird; the client renders it" | directly supported |
| "The out-of-scope section says rejected surfaces are not 'later' and that the architectural absence is the defense." | PLAN 1.2: "These are not 'later'" and "architectural absence is the defense" | directly supported |
| "Accessibility is a first-class designed surface, not a checklist fallback." | PLAN 8: "Designed for charm, not parity-by-checklist" and PLAN 10.1 launch gates | directly supported |

## Verdict

PASS.

The frozen reconstruction reads as plan-derived: no gold IDs, no scorer-side vocabulary, no gold-heading mirror, and no neat 1:1 mapping to the held-out target list. Its distinctive phrases overwhelmingly come directly from PLAN.md, and spot-checked articulate claims have clear plan support.
