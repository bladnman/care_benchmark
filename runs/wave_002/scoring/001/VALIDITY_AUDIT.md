# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

**Result: PASS.** The frozen reconstruction does not use gold-side IDs such as `S1`-`S9`, `F1`-`F40`, or `R-F01`. It uses plan-section numbering (`1. Scope`, `2. Architecture`, etc.) and plan/risk labels such as "Risk 7", which are present in `PLAN.md`.

No offending gold-ID hits were found in `RECONSTRUCTION.md`. The phrase `NOT RECOVERABLE FROM PLAN` appears repeatedly, but it is a phase-2A recovery convention rather than a gold-list identifier, and it does not map to specific gold IDs.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Judgment |
|---|---:|---|
| "The server is the only source of truth for simulation state" | yes | Directly plan-derived. |
| "Naturalist voice over announcement voice" | yes in substance | PLAN uses naturalist prose and rejects announcement-style narration; phrasing compresses plan material. |
| "Notice, never announce" | yes | PLAN Risk 7 names this directly. |
| "Privacy by architecture, not only policy" | yes in substance | PLAN says analytics has no read path to simulation DB and calls this "an architectural boundary, not a policy one." |
| "calming, not machine-like" | yes | Directly plan-derived from the loading sequence. |
| "slow monotonic-toward-expressive model" | yes | Directly plan-derived from scope and drift sections. |
| "care-taking pressure" | no exact phrase | Inference from PLAN's Tamagotchi non-goal; not gold/rubric vocabulary. |
| "demand-pull only" | yes | Directly plan-derived from visit-log scope. |
| "not a nice to have that ships in v1.1" | yes | Directly plan-derived from the accessibility risk mitigation. |
| "NOT RECOVERABLE FROM PLAN" | no | Reconstruction convention; not a gold-list phrase and not tied to gold IDs. |

I did not find rubric-side phrases such as "multi-layer recovery", "feature-level fidelity", "weight-3", "load-bearing", or "intent fidelity" in the reconstruction.

## Heading Mirror Check

The reconstruction headings are:

| Reconstruction heading | Comparison to gold headings | Judgment |
|---|---|---|
| `## System-level intent` | Generic phase-2A section, not a gold heading. | OK |
| `## Per-feature whys` | Generic phase-2A section, not a gold heading. | OK |
| `### 1. Scope` | Mirrors the PLAN section, not the gold list. | OK |
| `### 2. Architecture` | Mirrors the PLAN section, not the gold list. | OK |
| `### 3. Data model` | Mirrors the PLAN section, not the gold list. | OK |
| `### 4. API surface` | Mirrors the PLAN section, not the gold list. | OK |
| `### 5. Simulation engine design` | Mirrors the PLAN section, not the gold list. | OK |
| `### 6. Sync model` | Mirrors the PLAN section, not the gold list. | OK |
| `### 7. Frontend rendering pipeline` | Mirrors the PLAN section, not the gold list. | OK |
| `### 8. Audio pipeline` | Mirrors the PLAN section, not the gold list. | OK |
| `### 9. Accessibility surfaces` | Mirrors the PLAN section, not the gold list. | OK |
| `### 10. Performance budgets and observability` | Mirrors the PLAN section, not the gold list. | OK |
| `### 11. Rollout, risks, and appendix decisions` | Condenses later PLAN sections. | OK |

No reconstruction heading is an exact or near-exact mirror of the gold list's S/F why titles.

## 1:1 Mapping Suspect Check

**Result: not suspect.** The reconstruction does not provide neat S1-S9 and F1-F40 items in gold order. Instead it follows the candidate PLAN's own structure: a system summary, then plan-section-based feature bullets. It includes many features outside the 40 gold-why anchors and marks some plan bullets as `NOT RECOVERABLE FROM PLAN`. This reads like a reconstruction from `PLAN.md`, not a reconstruction from `GOLD_WHYS.md`.

## Plan-Derivation Spot Check

1. Reconstruction: "The server is the only source of truth for simulation state."
   PLAN support: `PLAN.md` says, "The server is the only source of truth for simulation state. Clients render and submit events; they never compute simulation outputs."
   Judgment: supported.

2. Reconstruction: "The product is either the aviary or the quiet field."
   PLAN support: `PLAN.md` loading sequence says, "The product is either the aviary or the quiet field."
   Judgment: supported.

3. Reconstruction: "The analytics warehouse has no read path to the simulation database."
   PLAN support: `PLAN.md` observability section says, "The analytics warehouse has no read path to the simulation database. This is an architectural boundary, not a policy one."
   Judgment: supported.

## Verdict

**PASS.** I found no significant contamination signatures. The reconstruction uses PLAN section structure, PLAN vocabulary, and PLAN risk language; it does not leak gold IDs, does not mirror the gold order, and its most articulate claims have direct PLAN support. A few phrases are inferential rather than exact, but they do not look gold- or rubric-derived.
