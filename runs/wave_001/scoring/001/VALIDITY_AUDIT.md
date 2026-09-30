# VALIDITY_AUDIT - run 001

## ID Leakage Check

Verdict for this section: PASS.

Mechanical search found no tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+` in either the frozen reconstruction or the assigned plan. The reconstruction never uses gold why IDs, rubric IDs, or an external scoring taxonomy.

## Vocabulary Check

Sampled reconstruction phrases against the PLAN:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `persistent identities` | PLAN §1: `persistent identities` | Exact plan vocabulary. |
| `first meaningful frame shows an ongoing place` | PLAN §1: same phrase | Exact plan vocabulary. |
| `same birds and vectors` | PLAN §11: recovery restores the same IDs/vectors | Plan-derived. |
| `never welcomes the user with a banner` | PLAN §1: same phrase | Exact plan vocabulary. |
| `matter-of-fact voice` | PLAN §5 and §10 use matter-of-fact system language | Plan-derived. |
| `ship together` | PLAN §1 says visual/audio/narration/caption/reduced-motion ship together | Exact plan vocabulary. |
| `not cycling three canned animations` | PLAN §6 uses this phrase | Exact plan vocabulary. |
| `simulation inputs, not a warehouse` | PLAN §11 uses this phrase | Exact plan vocabulary. |
| `first bird at actual renderer draw` | PLAN §12 uses this phrase | Exact plan vocabulary. |
| `never decrement prior drift` | PLAN §13 uses this phrase | Exact plan vocabulary. |

No suspicious scorer-side phrases such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, or `intent fidelity` appeared in the reconstruction.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Outcome, scope, and product invariants`
- `### Decisions where the plan needs interpretation`
- `### Architecture and ownership`
- `### Persistent data model`
- `### HTTP API and command contracts`
- `### Simulation engine and calibration`
- `### Presence, sync and disconnect behavior`
- `### Scene and interaction rendering`
- `### Procedural audio and captions`
- `### Accessibility and voice implementation`
- `### Privacy, security and lifecycle`
- `### Performance budgets and observability`
- `### Implementation sequence and acceptance evidence`
- `### Required verification matrix and risks`

Only the first two headings are required by phase-2A instructions. The remaining headings mirror the PLAN's own section structure, not the held-out gold list. They do not mirror `System-level whys`, `Feature-level whys`, S1-S9 names, or F1-F40 names.

## 1:1 Mapping Suspect Check

Verdict for this section: PASS.

The reconstruction does not create a neat S1-S9/F1-F40 list, does not use IDs, and does not proceed in gold-list order. Instead it has 12 broad system principles and a long set of per-feature rationales grouped by the plan's sections. The feature list is more granular than the 40 gold whys and includes many implementation/API/data-model items that are not gold why rows. This reads as plan-derived rather than target-list-derived.

## Plan-Derivation Spot Check

1. Reconstruction sentence: `The browser renders, interpolates, presents accessibility, and synthesizes calls, but it cannot decide canonical perch moves, mood changes, weather, greeting selection, acceptance of offers or drift.`
   PLAN support: §3 says the browser owns rendering/interpolation/accessibility/synthesis and `cannot decide canonical perch moves, mood changes, weather, greeting selection, acceptance of offers or drift`.
   Assessment: Directly plan-derived.

2. Reconstruction sentence: `Five-minute activity window and three-condition presence gate: The rationale is strict attention calibration with visibility, focus, and recent activity, rather than tab-open duration or broadened eligibility.`
   PLAN support: §7 chooses a five-minute window, evaluates visibility/focus/recent pointer-key activity, and says not to broaden eligibility to a tab-open check.
   Assessment: Directly plan-derived.

3. Reconstruction sentence: `Accessibility is a first-class deliverable, not a later adaptation.`
   PLAN support: §1 says visual/audio/narration/caption/reduced-motion experiences ship together; §13 says not to postpone accessible architecture until art completion.
   Assessment: Plan-derived summary, not gold-side leakage.

## Verdict: PASS

The frozen reconstruction shows no significant contamination signatures. It avoids gold IDs and scorer vocabulary, follows the plan's section order, and its articulate claims are traceable to PLAN passages. The reconstruction is valid for scoring as a plan-derived artifact.
