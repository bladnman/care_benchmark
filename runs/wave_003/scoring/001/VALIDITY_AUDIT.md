# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: no gold-ID leakage found. A targeted search for `S1`-`S9`, `F1`-`F40`, `R-F*`, rubric terms, and related tokens found no gold IDs used as gold IDs in the reconstruction. The only apparent `S3` hit was `S3-compatible bucket`, which also appears in PLAN and is not a benchmark identifier.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| `load-bearing rules` | PLAN intro names `load-bearing affective rules` | Plan-derived |
| `concrete enforcement points` | PLAN intro uses this exact phrase | Plan-derived |
| `single canonical aviary` | PLAN §7.1 heading and text | Plan-derived |
| `rule-of-the-product review` | PLAN §13.5 | Plan-derived |
| `quiet field` | PLAN §8.2 and §8.8 | Plan-derived |
| `aggregate-only operational telemetry` | PLAN §1.1, §11.6, §13.2 | Plan-derived |
| `not parity-by-checklist` | PLAN §1.1 and accessibility framing | Plan-derived |
| `no last-write-wins` | PLAN §7.6 | Plan-derived |
| `affective spine` | PLAN §14.1 says audio is the affective spine | Plan-derived |

Rubric-side terms such as `multi-layer recovery`, `feature-level fidelity`, `intent fidelity`, and `weight-3` do not appear in the reconstruction.

## Heading Mirror Check

Reconstruction headings: `System-level intent`, `Per-feature whys`, then PLAN-like sections: `Scope`, `Architecture`, `Data Model`, `API Surface`, `Auth`, `Simulation Engine`, `Sync Model`, `Frontend Rendering Pipeline`, `Audio Pipeline`, `Accessibility Surfaces`, `Performance Budgets and Observability`, `Voice and Content Discipline`, `Privacy, Compliance, and Engineering Boundaries`, `Rollout Plan`, `Risks, Explicit Calls, and Definition of Done`.

These mirror the PLAN's implementation sections, not GOLD_WHYS section titles or S/F ordering. `System-level intent` and `Per-feature whys` are expected phase-2A reconstruction headings, not suspicious gold-list mirroring.

## 1:1 Mapping Suspect Check

No 1:1 mapping to S1-S9 and F1-F40 was observed. The reconstruction has many more than 49 bullets, grouped by the PLAN's sections and implementation surfaces. It does not enumerate S IDs or F IDs, and it does not proceed in the gold list order. The mapping is broad and plan-structured, not gold-structured.

## Plan-Derivation Spot Check

1. Reconstruction: `The client is a render+capture surface only` and server-owned state makes the aviary continue without the viewer.
   PLAN support: §2.2 says exactly that the client is render+capture only and the split makes `the aviary continues without the viewer` true.

2. Reconstruction: telemetry and simulation are separated so per-bird/account state is not reachable from telemetry.
   PLAN support: §3.9, §11.6, and §13.2 specify aggregate telemetry only, separate analytics database, and no account/bird identifiers.

3. Reconstruction: starter birds are `the birds that arrived`, not catalog/avatar configuration.
   PLAN support: §15.11 and §16 explicitly state the first encounter should not feel like configuring an avatar and uses the `birds that arrived` wording.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses PLAN vocabulary, PLAN headings, and PLAN-derived rationale; it does not leak gold IDs, rubric terms, or a neat S/F mapping. Scores can be treated as valid for this run.
