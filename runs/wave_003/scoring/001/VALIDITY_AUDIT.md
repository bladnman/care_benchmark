# VALIDITY_AUDIT - CARE run 001

## ID Leakage

**Result: PASS.** A regex check for `F#`, `S#`, `R-F#`, and `R-S#` identifiers in the frozen reconstruction returned no hits. The reconstruction does not use gold IDs or an external gold taxonomy.

## Vocabulary Check

No rubric-side phrases such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, or `intent fidelity` appear in the reconstruction. The repeated phrase `NOT RECOVERABLE FROM PLAN` is expected because it was required by the phase-2A prompt. The term `canonical` appears, but it is plan vocabulary, not scorer vocabulary.

Sampled reconstruction phrases and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "browser-only, single-screen aviary" | PLAN §1 opens with this boundary. | Plan-derived. |
| "no generic notification service for aviary activity" | PLAN §1 states this directly. | Plan-derived. |
| "The worker is the only writer" | PLAN §2 states worker-only authorship. | Plan-derived. |
| "No absence or neglect term subtracts" | PLAN §5 states this directly. | Plan-derived. |
| "Privacy by projection, separation, and omission" | PLAN §§2-3 describe omitted raw vectors, separated telemetry, and privacy exclusions. | Plan-derived synthesis. |
| "Accessibility is a release path" | PLAN §8 heading and release condition match this. | Plan-derived. |
| "Visits are sharing without agency transfer" | PLAN §§1,4,10 describe same host state with no visitor controls/events/presence. | Plan-derived synthesis. |

## Heading Mirror

The reconstruction headings mirror the PLAN structure, not the held-out gold list. Headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Product boundary and decisions`
- `### System shape and ownership`
- `### Persistent model and privacy boundaries`
- `### API contract and event ingestion`
- `### Simulation, drift, mood, and calls`
- `### Client scene and interaction rendering`
- `### Audio and captions`
- `### Accessibility as a release path`
- `### Performance and observability gates`
- `### Build sequence, verification, and rollout`
- `### Main risks and responses`

Only the first two headings are required by the phase-2A prompt. The rest are PLAN section headings. They do not mirror S1-S9, F1-F40, or rubric section names.

## 1:1 Mapping Suspect

**Result: not suspect.** The reconstruction does not enumerate S1-S9 or F1-F40 and does not follow the gold order. It follows the PLAN's sections and includes many implementation rows outside the gold-why list. The high item count is explained by the plan's own detailed structure, not by a neat target-list mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Canonical, server-authored life rather than client simulation." | PLAN §2 says the worker is the only writer and the browser owns only ephemeral rendering state. | Supported. |
| "Continuity without punishment." | PLAN §5 specifies nonnegative deltas, no absence/neglect subtraction, and two-week absence cases where birds are quieter, not harmed. | Supported. |
| "Accessibility is a release path, not a supplement." | PLAN §8 is titled "Accessibility as a release path" and says the release condition is a continuing place, not just reachable controls. | Supported. |

## Verdict

**PASS.** The reconstruction reads as a faithful, plan-derived summary. It contains no gold ID leakage, no scorer-side vocabulary, no gold-list heading mirror, and no 1:1 gold mapping. Its headings and vocabulary track the assigned PLAN, and sampled articulate claims are supported by PLAN passages.
