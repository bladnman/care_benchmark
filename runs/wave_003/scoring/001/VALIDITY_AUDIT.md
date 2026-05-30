# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS. I found no gold IDs or rubric-style IDs in the frozen reconstruction. The reconstruction headings and bullets do not use S1-S9, F1-F40, R-F identifiers, or comparable gold-taxonomy labels. The PLAN also does not expose those IDs, so there is no ID-leakage hit to cross-reference.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Canonical aviary state is server-owned end to end" | PLAN line 7 | Directly plan-derived |
| "immediate transient scene reactions" | PLAN lines 8 and 225 | Directly plan-derived |
| "Snapshot delivery is pull-based" | PLAN line 9 | Directly plan-derived |
| "deterministic product logic, not an external LLM" | PLAN line 10 | Directly plan-derived |
| "continuity, specificity, and restraint" | PLAN line 629 | Directly plan-derived |
| "presence is the dominant drift input" | PLAN line 232 | Directly plan-derived |
| "physically separated from per-account simulation tables" | PLAN line 47 | Directly plan-derived |
| "first-class product surfaces" | PLAN line 25 | Directly plan-derived |
| "visible same-day jumps" | PLAN line 252 | Directly plan-derived |
| "track mixer" | PLAN line 452 | Directly plan-derived |

I did not find suspicious rubric-side phrases such as "multi-layer recovery", "feature-level fidelity", "weight-3", "intent fidelity", or gold-only labels in the reconstruction. The phrase "NOT RECOVERABLE FROM PLAN" appears as part of the expected reconstruction output convention, not as gold-list contamination.

## Heading Mirror Check

| Reconstruction heading | Closest source | Assessment |
|---|---|---|
| System-level intent | Phase-2A expected structure | Not a gold-list mirror |
| Per-feature whys | Phase-2A expected structure | Not a gold-list mirror |
| Scope | PLAN heading | Plan-derived |
| Architecture and data model | PLAN headings | Plan-derived compression |
| API surface and polling model | PLAN headings | Plan-derived compression |
| Simulation engine design | PLAN heading | Plan-derived |
| Frontend rendering pipeline | PLAN heading | Plan-derived |
| Audio pipeline | PLAN heading | Plan-derived |
| Accessibility surfaces | PLAN heading | Plan-derived |
| Performance budgets, observability, and rollout | PLAN headings | Plan-derived compression |

The headings do not mirror GOLD_WHYS section titles such as "System-level whys" or named S/F entries. They mostly mirror or compress PLAN sections.

## 1:1 Mapping Suspect Check

Verdict for this check: PASS. The reconstruction does not provide a neat S1-S9 plus F1-F40 list in gold order. It has 10 system-level bullets and a plan-structured implementation walk-through. The feature bullets follow PLAN sections, include many non-gold implementation elements, and mark some plan items as not recoverable. This is not a 1:1 gold mapping.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Product voice should be deterministic, private, and consistent."
   PLAN support: line 10 says notebook prose, captions, and narration come from deterministic product logic to keep voice consistent, latency low, and private bird history out of third-party systems.
   Assessment: plan-derived.

2. Reconstruction sentence: "Render pipeline boundary: keeping 'what is true' on the server and 'how to draw that truth' on the client preserves the 'aviary continues without the viewer' illusion while preventing state divergence."
   PLAN support: lines 77-81 make the same server/client truth split and name the continuing-without-viewer illusion and state-divergence prevention.
   Assessment: plan-derived.

3. Reconstruction sentence: "Presence accounting with visibility, focus, recent activity, clamping, and device-window union: presence is 'the dominant drift input' and 'must remain honest.'"
   PLAN support: lines 232-237 specify visible/focused/recently active pings, clamping, device windows, and unioning overlapping attention.
   Assessment: plan-derived.

## Verdict

PASS. The reconstruction reads as derived from PLAN structure and vocabulary, with no significant signs of gold-list or rubric contamination. It is sometimes too close to the plan's mechanisms and sometimes under-recovers the gold rationale, but that is an ordinary reconstruction limitation rather than a contamination signature.
