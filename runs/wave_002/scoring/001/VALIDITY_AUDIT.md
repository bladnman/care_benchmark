# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The reconstruction reads as plan-derived. I found no gold ID leakage, no rubric-side taxonomy, no near-1:1 mirror of the gold list, and no unsupported articulate sentence in the spot checks. The only suspicious vocabulary hit was "load-bearing," which appears in the PLAN itself.

## ID Leakage

Mechanical check searched for gold-style IDs such as `F1`, `F40`, `S1`, `S9`, `R-F01`, and `R-S01` in `RECONSTRUCTION.md`.

- Hits: none.
- Cross-check: not needed because no offending ID appeared.

The reconstruction uses plan section terms such as M0/M1/M3 for milestones, but those are plan-native milestone labels, not gold IDs.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "structural or automated constraint" | yes, PLAN §0 | Plan-derived |
| "rot silently" | yes, PLAN §0 | Plan-derived |
| "Notice, never announce" | yes, repeated in PLAN | Plan-derived |
| "the relationship" | yes, in trait/export/privacy contexts | Plan-derived |
| "three-signal conjunction" | yes, PLAN §5.5 and §12.1 | Plan-derived |
| "architectural absence" | near match: PLAN uses structural absences and absence of routes/tables | Plan-derived paraphrase |
| "load-bearing decision" | yes, PLAN §2.1 | Plan-derived; not a contamination hit |
| "deliberately boring" | yes, PLAN §2.1 | Plan-derived |
| "charm-invariant suite" | yes, PLAN §12 | Plan-derived |
| "feature-level fidelity" / "multi-layer recovery" / "weight-3" | no hits in reconstruction | No rubric vocabulary leakage |

## Heading Mirror

Top-level reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`

Those headings match the phase-2A prompt shape, not the gold list. Lower headings mirror the PLAN's implementation sections: Scope boundaries and non-goals, Accounts and identity, The aviary, The bird engine, Interactions, Sync and social, API and data mechanics, Simulation engine, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Performance and observability, Rollout/risk/success posture. They do not mirror GOLD_WHYS section titles or S/F ordering.

## 1:1 Mapping Suspect

No. The reconstruction does not create neat S1-S9 or F1-F40 rows, does not use gold IDs, and does not follow the gold list order. It reconstructs many more plan-native items than the 49 scored whys and groups them by the plan's own structure.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The plan's organizing principle says every affective rule in the PRD is translated into a structural or automated constraint, because affective requirements rot silently."
   - PLAN support: §0 says "every affective rule... translated into a structural or automated constraint" and "Affective requirements rot silently."
   - Assessment: supported.

2. Reconstruction sentence: "The visitor path has no event-ingest routes and no write grants, so visitor attention cannot be recorded as drift input."
   - PLAN support: §4.5 says the visitor router has no event-ingest routes and a SELECT-only database role; §3.5 says there is no visitor presence/event path.
   - Assessment: supported.

3. Reconstruction sentence: "Reduced motion uses the same scene graph and state, narration shares the naturalist grammar package, and M3 is a launch gate rather than a later fix."
   - PLAN support: §7.6 says reduced motion uses the same scene graph and state; §9.1 says narration shares naturalist grammar with notebook; §11.1 makes M3 an accessibility gate.
   - Assessment: supported.

## Verdict Rationale

The reconstruction uses the PLAN's vocabulary, sectioning, and implementation emphasis. It contains plan-native milestone labels and implementation concepts, not gold IDs or scorer vocabulary. Minor vocabulary overlap with rubric-sounding words is explained by the PLAN itself, so the audit passes.
