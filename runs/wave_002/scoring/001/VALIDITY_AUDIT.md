# VALIDITY_AUDIT - CARE run 001

## ID Leakage

PASS. Mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. The reconstruction uses plan-local numbering: `### 1. Scope`, `### 2. Architecture`, and so on.

## Vocabulary Check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled phrases and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Quiet, non-gamified relationship" | PLAN.md §1.2 excludes gamification, toasts, social network surfaces; §20 says "neglect is quiet ambient, never guilt". | Plan-derived synthesis. |
| "Server-owned, canonical personality" | PLAN.md §1.1 says server tick is sole writer; §6.1 says only sim-worker updates personality columns. | Plan-derived synthesis. |
| "Monotonic, slow, attention-shaped drift" | PLAN.md §5.2 has upward-only deltas, no decrease on neglect, 7/21 day calibration. | Plan-derived synthesis. |
| "Alive, not loading theater" | PLAN.md §7.3 heading and content describe mid-pose first paint, no fade, no spinner. | Directly plan-derived. |
| "Naturalist voice for the aviary; matter-of-fact voice for systems" | PLAN.md §14 voice lint, §4.1 matter-of-fact error bodies, §11 naturalist notebook prose. | Plan-derived. |
| "Accessibility is part of the affective product" | PLAN.md §15.5 requires reduced-motion and screen-reader story as "charm, not bare labels". | Plan-derived synthesis. |
| "hard architecture boundaries" for privacy | PLAN.md §2.1 says no analytics warehouse read path into per-bird tables; §12.3 defines a hard privacy line. | Plan-derived synthesis. |

## Heading Mirror

The only `##` headings are `System-level intent` and `Per-feature whys`, which were required by the phase-2A reconstruction prompt. The `###` headings mirror PLAN.md section order (Scope, Architecture, Data model, API surface, Simulation engine design, Sync model, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Presence accounting, Field notebook generation, Performance budgets and observability, Privacy implementation, Frontend modules map, Rollout, Risks and mitigations, Testing strategy, Security notes, Document collapse of "what not to build" into eng checklist, Success definition). They do not mirror GOLD_WHYS.md sections or S/F IDs.

## 1:1 Mapping Suspect

PASS. The reconstruction does not provide a neat S1-S9/F1-F40 list or gold-order mapping. It follows the PLAN.md structure and expands many plan bullets, including numerous items that are not canonical gold why rows. Several entries are honestly marked `NOT RECOVERABLE FROM PLAN`, which is inconsistent with a contaminated 1:1 gold mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan's drift rule is upward only: 'Neglect -> Delta t = 0 (no decrease).'" | PLAN.md §5.2 states "Neglect -> Delta t = 0 (no decrease)". | Direct support. |
| "The first paint path asks for 'soft sky CSS immediately,' a bird drawn 'mid-pose' with 'no fade-from-static, no entry animation,' and 'quiet field only' if slow, 'never spinner.'" | PLAN.md §7.3 contains these first-paint and slow-snapshot loading-state rules. | Direct support. |
| "Visits are described as 'Quiet visit invitations,' 'opt-in per invite,' 'read-only ambient,' 'revocable,' and 'default off.'" | PLAN.md §1.1 includes those visit scope terms; §4.5 defines invite/revoke/view/snapshot routes. | Direct support. |

## Verdict

PASS. The reconstruction reads as derived from the assigned PLAN.md: no gold IDs, no rubric vocabulary, headings mirror plan sections rather than gold sections, and articulate sentences can be grounded in exact plan passages. The reconstruction is valid for scoring.
