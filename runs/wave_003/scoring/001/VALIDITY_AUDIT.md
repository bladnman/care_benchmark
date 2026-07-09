# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found.

A targeted search of the frozen reconstruction found no gold IDs such as `S1`, `F1`, `F40`, or `R-F01`. The same targeted search in PLAN also found no such IDs. There are no offending reconstruction sentences to list.

## Vocabulary Check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "continuing place, not a loading or restart story" | PLAN.md:5 says the first visible frame must read as a continuing place; PLAN.md:278-282 specifies quiet field/no spinner/mid-action bird. | Plan-derived. |
| "server-authoritative continuity" | PLAN.md:9 says the server is sole authority; PLAN.md:360 says only the tick transaction writes personality/mood. | Plan-derived. |
| "honest presence as a trusted input" | PLAN.md:12 and :214 define the three-signal presence requirement. | Plan-derived. |
| "quiet anti-gamification" | PLAN.md:20 rejects streak/score/status/public-discovery changes; PLAN.md:40 lists excluded gamification surfaces. | Plan-derived summary. |
| "Privacy by schema, pipeline, and product surface" | PLAN.md:18, :57, :59, and :381-393 define aggregate telemetry and separation. | Plan-derived summary. |
| "Accessible modes are first-class, not substitutes" | PLAN.md:17 and :356 require complete accessible modes and block release on regression. | Plan-derived, not rubric-specific. |
| "NOT RECOVERABLE FROM PLAN" | This phrase is part of the reconstruction task style, not a gold/rubric taxonomy. | Not contamination. |
| "recovery" | Appears only in ordinary product/account contexts such as deletion recovery and snapshot recovery. | Not rubric vocabulary use. |

No suspicious rubric-side terms such as "multi-layer recovery", "feature-level fidelity", "weight-3", "gold why", or "intent fidelity" appear in the reconstruction.

## Heading Mirror Check

| Reconstruction heading | Closest relation | Assessment |
|---|---|---|
| `## System-level intent` | Required reconstruction structure, not a gold-list title. | Acceptable. |
| `## Per-feature whys` | Required reconstruction structure, not mirrored to S/F IDs. | Acceptable. |
| `### V1 scope` | Mirrors PLAN.md section 2. | Plan-derived. |
| `### Proposed system shape` | Mirrors PLAN.md section 3. | Plan-derived. |
| `### Data model and retention` | Mirrors PLAN.md section 4. | Plan-derived. |
| `### API contracts` | Mirrors PLAN.md section 5. | Plan-derived. |
| `### Simulation engine` | Mirrors PLAN.md section 6. | Plan-derived. |
| `### Client rendering and interaction pipeline` | Mirrors PLAN.md section 7. | Plan-derived. |
| `### Procedural audio` | Mirrors PLAN.md section 8. | Plan-derived. |
| `### Accessibility implementation` | Mirrors PLAN.md section 9. | Plan-derived. |
| `### Sync, consistency, and failure handling` | Mirrors PLAN.md section 10. | Plan-derived. |
| `### Performance budgets and observability` | Mirrors PLAN.md section 11. | Plan-derived. |
| `### Verification strategy` | Mirrors PLAN.md section 12. | Plan-derived. |
| `### Work breakdown, rollout, and open decisions` | Condenses PLAN.md sections 13-16. | Plan-derived. |

The headings mirror PLAN sections rather than GOLD_WHYS system or feature titles. No near-exact gold-list section title mirror was found.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not present neat S1-S9 or F1-F40 items, does not use gold IDs, and does not follow the gold-list order. Instead it follows the PLAN's implementation sections and produces many more than 49 bullets. This is consistent with blind plan-derived reconstruction, not gold-list contamination.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan repeatedly says the server is the 'sole authority' and that clients 'submit facts and render derived state.'" | PLAN.md:9 contains both phrases. | Directly grounded. |
| "A qualifying interval starts only while visible, focused, and recently active, preserving presence as a trusted input." | PLAN.md:214 says a qualifying interval begins only while visible + focused + recent activity. | Directly grounded. |
| "Reduced motion must remain alive through 'still-pose cross-fades and color/audio,' and accessibility regressions block release like a broken visual scene." | PLAN.md:350 specifies still-pose cross-fades and retained color changes; PLAN.md:356 says accessibility regressions block release like a broken visual scene. | Directly grounded. |

## Verdict

PASS. The reconstruction reads as plan-derived: it mirrors PLAN headings, uses no gold IDs, avoids rubric vocabulary, and its most articulate claims trace back to exact PLAN passages. The main issues are reconstruction omissions and a few false `NOT RECOVERABLE FROM PLAN` statements, not contamination.
