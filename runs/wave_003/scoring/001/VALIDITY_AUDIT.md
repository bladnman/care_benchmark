# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: PASS. Mechanical search found no gold IDs or rebuild IDs in the frozen reconstruction: no `F1`-style, `S1`-style, `R-F01`, or `R-S01` tokens appeared. Because there were no hits, no PLAN cross-reference leakage check was needed.

## Vocabulary Check

Mechanical search found no scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing`.

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "one canonical, server-owned aviary" | PLAN line 5 | Plan-derived |
| "already alive on first render" | PLAN line 5 | Plan-derived |
| "No feature should manufacture urgency around absence" | PLAN line 23 | Directly plan-derived |
| "hidden server-only normalized values" | PLAN line 68 | Plan-derived |
| "lower-case, present-tense, specific naturalist observations" | PLAN line 20 | Directly plan-derived |
| "no warehouse replication or query path" | PLAN line 27 | Directly plan-derived |
| "parallel primary experience" | PLAN line 238 | Directly plan-derived |
| "versioned simulation module" | PLAN line 49 | Directly plan-derived |
| "v1 scope failure" | PLAN line 333 | Directly plan-derived |

## Heading Mirror

The reconstruction headings mirror the assigned PLAN structure, not the gold list. It has the required `## System-level intent` and `## Per-feature whys`, then plan-section headings such as "Delivery contract and scope guardrails," "System shape and bounded responsibilities," "Persistent model, invariants, and retention," and "Definition of done." These do not mirror the gold-list S1-S9/F1-F40 taxonomy or file-group headings. Assessment: PASS.

## 1:1 Mapping Suspect

The reconstruction does not provide a neat S1-S9 or F1-F40 sequence. It gives 10 system-level principles and then many plan-section bullets in the same rough order as PLAN.md. The order and grouping are plan-derived and do not match the 49 gold targets. Assessment: PASS.

## Plan-Derivation Spot Check

1. Reconstruction: "The client 'never owns or computes durable bird personality'; it submits facts and renders an authoritative snapshot." PLAN support: line 5 says the client submits facts and never owns durable bird personality; lines 33-39 and 45 reinforce snapshot rendering and server ownership. Result: plan-derived.
2. Reconstruction: "A slow snapshot should show 'a quiet field with restrained ambient cues,' never a spinner, skeleton, toast, wake-up animation, or static placeholder presented as the aviary." PLAN support: line 53 uses the same quiet-field/no-spinner language. Result: plan-derived.
3. Reconstruction: "Overcounting inactive tabs or spoofed presence corrupts personality and trust, so presence requires visible + focused + recent activity." PLAN support: lines 142-144 define the visible/focused/recent activity conjunction and fail-closed behavior; line 316 says overcount corrupts personality and trust. Result: plan-derived.

## Verdict

PASS. The frozen reconstruction reads as a faithful plan-derived reconstruction: no gold IDs, no rubric vocabulary, no gold-order mapping, and its headings and articulate claims are traceable to PLAN.md. I found no contamination signatures that should make this run's scores suspect.
