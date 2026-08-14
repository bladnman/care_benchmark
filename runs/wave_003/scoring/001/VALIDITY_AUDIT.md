# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS

The frozen reconstruction reads as plan-derived. It contains no leaked S/F/R-F/R-S gold IDs, no rubric-side scoring vocabulary, and its headings follow the PLAN's own section structure. Some phrasing is polished and compressed, but the supporting passages are traceable to the PLAN.

## ID Leakage

Mechanical grep for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` found no hits in `RECONSTRUCTION.md`. Because there are no hits, there is no ID leakage to cross-check against `PLAN.md`.

## Vocabulary Check

Rubric/gold-side terms checked: `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, `intent fidelity`, `GOLD_WHYS`, and `RUBRIC`. None appeared. The word `canonical` appears in the reconstruction, but it is plan-derived: the PLAN repeatedly calls state and the simulation tick canonical.

Plan-derived phrase samples:

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| "low-key, observational relationships" | PLAN.md:8 | Directly plan-derived |
| "Notice, never announce" | PLAN.md:10 | Directly plan-derived |
| "zero decay or negative drift on neglect" | PLAN.md:21 | Directly plan-derived |
| "3-factor presence condition" | PLAN.md:566-591 | Plan-derived paraphrase |
| "canonical heart of the product" | PLAN.md:437-439 | Directly plan-derived |
| "Privacy & Telemetry Firewall" | PLAN.md:114-121 | Directly plan-derived |
| "Matter-of-Fact Register" | PLAN.md:52-56 | Directly plan-derived |
| "Screen-Reader Queue Saturation" | PLAN.md:882-884 | Directly plan-derived |

## Heading Mirror

`RECONSTRUCTION.md` headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Executive Summary & Scope`
- `### System Architecture & Boundaries`
- `### Data Model & Database Schemas`
- `### API Surface & Protocols`
- `### Simulation Engine Design`
- `### State Synchronization & Concurrency Model`
- `### Frontend Rendering Pipeline`
- `### Audio Engine & Procedural Synthesis`
- `### Accessibility Surfaces`
- `### Performance Budgets, Optimization & Observability`
- `### Phased Rollout & Lifecycle Strategy`
- `### Risk Management & Engineering Mitigations`

These mirror the PLAN headings, not the gold-list structure. They do not mirror S1-S9, F1-F40, or gold file-group headings in a suspect way.

## 1:1 Mapping Suspect

Not suspect. The reconstruction has 11 system-level bullets and a PLAN-section-oriented per-feature pass; it does not enumerate exactly S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold order. It also marks some features `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction behavior.

## Plan-Derivation Spot Check

1. Reconstruction: "The plan defines attention through a strict `presence` condition and discrete interactions."  
   PLAN support: PLAN.md:566-591 defines visibility, focus, and recent user activity as the presence detector.

2. Reconstruction: "The simulation tick is called the `canonical heart of the product`."  
   PLAN support: PLAN.md:437-439 says the simulation tick is the canonical heart and runs server-side every 60 seconds.

3. Reconstruction: "The plan bypasses failed audio setup, activates captions, and avoids canned fallback files to protect bundle size."  
   PLAN support: PLAN.md:748-752 says failed WebAudio activates captions and loads no canned audio or fallback files.

All three articulate sentences are directly supported by the PLAN.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction appears to have been derived from `PLAN.md` rather than from the held-out gold/rubric materials. Scores are not treated as suspect on validity grounds.
