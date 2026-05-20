# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS** - The reconstruction reads as derived from PLAN.md rather than from the gold list or rubric. It uses PLAN section structure, does not leak gold IDs, and does not map neatly onto S1-S9/F1-F40.

## ID leakage

No gold IDs or rubric-style identifiers appear in the frozen reconstruction. I found no uses of S1-S9, F1-F40, R-F01, canonical feature IDs, weight labels, or score-taxonomy IDs. Because there are no hits in RECONSTRUCTION.md, there is no PLAN cross-reference leakage to report.

## Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| observer-focused naturalist experience | PLAN 1.1 says Naturalist Experience and observer-focused interaction | plan-derived |
| monotonic positive drift towards expressiveness | PLAN 1.2 and 5.2 describe monotonic drift and values never decreasing | plan-derived |
| server-canonical | PLAN 1.1 and 6 say server-canonical synchronization | plan-derived |
| pure rendering and audio synthesis engine | PLAN 2 says the client is a pure rendering and audio synthesis engine | plan-derived |
| append-only event log | PLAN architecture and schema use Append-Only Event Log | plan-derived |
| Matter-of-fact tone on error | PLAN 4.1 says matter-of-fact tone on error | plan-derived |
| Accessibility is treated as a core product feature | PLAN 9 states this directly | plan-derived |
| No co-presence or guest-driven drift | PLAN 1.1 visits description states this directly | plan-derived |
| measurable instrument change / visible user difference | PLAN 5.2 calibration target states these phrases | plan-derived |
| NOT RECOVERABLE FROM PLAN | A reconstruction convention, not gold/rubric-specific scoring vocabulary | acceptable |

I found no suspicious free use of rubric-side phrases such as multi-layer recovery, feature-level fidelity, weight-3, intent fidelity, or gold why.

## Heading Mirror

RECONSTRUCTION.md headings mirror PLAN.md sections: Scope & Non-Goals, Architecture & Service Shape, Data Model, API Surface, Simulation Engine Design, Sync & Consistency Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, Performance Budgets & Observability, Rollout & Tuning, and Risks & Mitigations. They do not mirror GOLD_WHYS.md sections or S/F ordering.

## 1:1 Mapping Suspect

No 1:1 mapping to gold targets is present. The reconstruction does not enumerate S1-S9 or F1-F40, does not cover every gold target in gold order, and includes many PLAN-local implementation items that are not gold whys. The mapping is to PLAN sections and bullets, which is expected for a plan-derived reconstruction.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The simulation is server-canonical, while clients render, synthesize audio, and submit events."
   - PLAN support: Section 2 says the server is the single source of truth and the client is a pure rendering and audio synthesis engine; Section 6 says clients submit events and never send absolute values.

2. Reconstruction sentence: "The product avoids punishment, distress, and obligation."
   - PLAN support: Section 1.2 says birds cannot die, fall ill, or show distress, neglect produces ambient quietness, and drift is monotonic positive.

3. Reconstruction sentence: "Accessibility is part of the core product surface, not a later accommodation."
   - PLAN support: Section 9 says accessibility is treated as a core product feature, and Section 12.4 requires accessibility changes to merge in the same pull request as visual updates.

## Verdict Rationale

**PASS.** The reconstruction is often compressed and sometimes overgeneral, but its wording and structure are anchored in PLAN.md. There is no evidence of gold ID leakage, rubric vocabulary leakage, heading mirroring, or a suspicious gold-order mapping.
