# VALIDITY_AUDIT — CARE run 001

## Verdict

**PASS.** The reconstruction reads as derived from the candidate PLAN rather than from the gold list or rubric. It mirrors the plan's section structure, uses no gold IDs, and its stronger phrases can be traced to plan wording or ordinary synthesis. The phrase `NOT RECOVERABLE FROM PLAN` is not in the plan, but it is a reconstruction convention rather than gold-side leakage.

## ID Leakage

No gold IDs were found in `RECONSTRUCTION.md`: no `S1`-`S9`, no `F1`-`F40`, and no `R-Fxx` identifiers. The same search in `PLAN.md` also found no such IDs, so there are no ID leakage hits to cross-reference.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `long-term relationships with procedural animated birds` | PLAN.md:3 has the same phrase. | Plan-derived. |
| `Tamagotchis` / `static wallpapers` | PLAN.md:154 uses both failure endpoints. | Plan-derived. |
| `Source of Truth` | PLAN.md:31 uses this phrase. | Plan-derived. |
| `Interaction Events, never absolute trait values` | PLAN.md:88 has the same rule. | Plan-derived. |
| `felt-aliveness` | PLAN.md:38 uses this exact word. | Plan-derived. |
| `Visibility + Focus + Activity` | PLAN.md:158 uses this exact conjunction. | Plan-derived. |
| `cross-cutting boundary` | Exact phrase is not in PLAN, but reconstruction line 11 synthesizes multiple privacy surfaces from PLAN.md:45-48, 147, and 15/23. | Minor synthesis; not gold-specific. |
| `NOT RECOVERABLE FROM PLAN` | Not in PLAN. | Expected reconstruction convention, not evidence of gold/rubric content. |
| `slow cadence` | PLAN.md:126 says narration is at a slow cadence. | Plan-derived. |
| `p99 > 5s` | PLAN.md:145 has the same alarm threshold. | Plan-derived. |

No rubric-only terms such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `intent fidelity`, or gold why IDs appear in the reconstruction.

## Heading Mirror

Reconstruction headings mirror the PLAN, not the gold list:

| Reconstruction heading | Closest PLAN heading | Gold-list mirror? |
|---|---|---|
| `## System-level intent` | Phase-2A output structure, not PLAN. | No direct gold section-title mirror beyond required reconstruction format. |
| `## Per-feature whys` | Phase-2A output structure. | No; required reconstruction format. |
| `### 1. Scope and v1 Boundaries` | PLAN.md `## 1. Scope and v1 Boundaries`. | No. |
| `### 2. Architecture` | PLAN.md `## 2. Architecture`. | No. |
| `### 3. Data Model` | PLAN.md `## 3. Data Model`. | No. |
| `### 4. Simulation Engine Design` | PLAN.md `## 4. Simulation Engine Design`. | No. |
| `### 5. Sync and Multi-Device Model` | PLAN.md `## 5. Sync & Multi-Device Model`. | No. |
| `### 6. API Surface` | PLAN.md `## 6. API Surface`. | No. |
| `### 7. Frontend Rendering and Audio` | PLAN.md `## 7. Frontend Rendering & Audio`. | No. |
| `### 8. Accessibility Surfaces` | PLAN.md `## 8. Accessibility Surfaces`. | No. |
| `### 9. Performance and Observability` | PLAN.md `## 9. Performance & Observability`. | No. |
| `### 10. Rollout and Risk` | PLAN.md `## 10. Rollout & Risk`. | No. |

## 1:1 Mapping Suspect

The reconstruction does not provide a neat S1-S9 / F1-F40 mapping in gold-list order. Instead, it creates seven system-level bullets and then walks the PLAN sections and plan bullets. This is a plan-derived mapping. It omits many gold targets entirely, marks several plan bullets as `NOT RECOVERABLE FROM PLAN`, and does not use the gold taxonomy.

## Plan-Derivation Spot Check

1. Reconstruction: `Canonical bird state is server-owned, event-driven, and conflict-resistant.`
   PLAN support: PLAN.md:31 says the server owns the `Source of Truth`; PLAN.md:87-92 says only the server writes personality vectors, clients submit interaction events, additive deltas are processed in event-log order, and last-write-wins is prohibited.
   Result: supported.

2. Reconstruction: `The plan separates canonical simulation from local felt detail.`
   PLAN support: PLAN.md:35-38 divides server simulation, client interpolation, and client ornaments, with ornaments preserving `felt-aliveness` without server overhead.
   Result: supported.

3. Reconstruction: `Privacy is a cross-cutting boundary, not just an account detail.`
   PLAN support: PLAN.md:45-46 uses synthetic account IDs and encrypted email; PLAN.md:15 makes visits opt-in; PLAN.md:23 refuses social-network surfaces; PLAN.md:147 excludes bird names, trait values, and interaction history from telemetry.
   Result: supported synthesis.

## Final Verdict

**PASS.** There are minor non-plan phrases, especially `cross-cutting boundary` and `NOT RECOVERABLE FROM PLAN`, but no gold IDs, no rubric-specific scoring vocabulary, and no gold-order mapping. The reconstruction is best read as a faithful plan-derived synthesis with some ordinary evaluator phrasing, not as contaminated by the gold list.
