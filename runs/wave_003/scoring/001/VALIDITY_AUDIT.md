# VALIDITY_AUDIT - CARE run 001

## ID Leakage

No gold IDs or rubric IDs were found in the frozen reconstruction. It does not use `S1`-`S9`, `F1`-`F40`, `R-F01`, weight labels, or feature-level scoring taxonomy. The only structured labels are the reconstruction headings and plan-section headings.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Core-first v1 prototype scope" | PLAN Scope says v1 focuses on "establishing the core" and has in/out-of-scope bullets. | Plan-derived paraphrase. |
| "synchronous aviary simulation" | Exact phrase appears in PLAN Scope. | Plan-derived. |
| "real-time, server-centered synchronization" | PLAN has server state synchronization, WebSockets for real-time updates, and canonical server state pushed to clients. | Synthesized from plan, not rubric-specific. |
| "bird behavior evolves through drift" | PLAN says "bird-evolution" and "personality drift". | Plan-derived. |
| "realistic evolution and predictable simulation" | Exact risk wording appears in PLAN Risks. | Plan-derived. |
| "responsive birdsong" | Exact phrase appears in PLAN Audio Pipeline. | Plan-derived. |
| "product surface" | PLAN does not use this phrase; it is a harmless reconstructor abstraction around the Accessibility section. | Minor paraphrase, not gold/rubric vocabulary. |
| "performance budgets shape implementation choices" | PLAN names tick frequency configurable by performance budget, CSS composition, bundle, 60fps, and sub-100ms targets. | Plan-derived synthesis. |
| "NOT RECOVERABLE FROM PLAN" | Not in PLAN; appears to be a reconstruction convention for absent rationales. | Not contamination; no gold content. |

No suspect rubric-side phrases such as "multi-layer recovery", "weight-3", "intent fidelity", "gold why", or "feature-level fidelity" appear in the reconstruction.

## Heading Mirror

Reconstruction headings mirror the PLAN structure, not the gold list. Top-level headings are `System-level intent` and `Per-feature whys`; subheadings are `Scope`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine Design`, `Sync Model`, `Rendering Pipeline`, `Audio Pipeline`, `Accessibility`, `Performance`, and `Risks`. These match the PLAN section headings. They do not mirror gold-list sections such as `System-level whys`, `Feature-level whys`, `concepts.md`, or the S/F entries.

## 1:1 Mapping Suspect

No 1:1 mapping to S1-S9 or F1-F40 is present. The reconstruction has six system-level bullets and then plan-section feature bullets in the same order as PLAN.md. It does not enumerate 9 system whys, 40 feature whys, or the canonical gold ordering. This pattern supports plan derivation rather than gold-list leakage.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Judgment |
|---|---|---|
| "The plan repeatedly carries "synchronous aviary simulation," "state synchronization," "real-time state updates," "event propagation," "delta updates," and "Canonical server state pushed to all clients." | Scope, Architecture, Communication, API Surface, and Sync Model contain these exact or near-exact phrases. | Supported. |
| "The `Risks` section makes the philosophy explicit as a balance between "realistic evolution and predictable simulation." | PLAN Risks: "Drift calibration: Difficulty finding the balance between realistic evolution and predictable simulation." | Supported. |
| "The plan calls for "Full screen-reader support via live regions for presence events and bird interactions" and "Captions for all procedural calls" in `Accessibility`." | PLAN Accessibility has those exact sentences. | Supported. |

## Verdict

**PASS.** The reconstruction reads as a plan-derived summary: it follows PLAN.md headings, uses plan phrases heavily, lacks gold IDs and rubric vocabulary, and does not show a neat S/F target mapping. The one non-plan phrase "product surface" is a generic abstraction and does not carry gold-list substance.
