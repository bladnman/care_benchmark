# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no ID leakage found. A regex search for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits in the frozen reconstruction. The reconstruction does not use gold IDs, rubric IDs, or external target-list numbering.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "watchful presence" | PLAN §1 includes "watchful presence" | Plan-derived |
| "ongoing aviary" | PLAN §1 says the first usable frame shows the ongoing aviary | Plan-derived |
| "server is the sole writer" | PLAN §2 uses this architecture | Plan-derived |
| "bounded intentions and attention evidence" | PLAN §2 uses this phrase | Plan-derived |
| "designed second renderer" | PLAN §6 uses this phrase for reduced motion | Plan-derived |
| "state dump" | PLAN §10 warns accessibility must not become a state dump | Plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Required phase-2A marker, not gold/rubric vocabulary | Not suspicious |
| "recovery" | Appears in ordinary account-deletion/retry contexts only | Not suspicious |

No scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` appear.

## Heading Mirror

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then `###` headings matching PLAN sections: Product contract and scope; Delivery shape and ownership; Persistent data model; API and authorization contract; Presence, event processing, and simulation; Scene and audio pipelines; Accessibility, interaction, and copy; Performance, privacy, and operations; Execution sequence and acceptance gates; Key risks and explicit decisions.

These mirror PLAN structure, not GOLD_WHYS structure. They do not match the gold feature groups by file or the S/F target list.

## 1:1 Mapping Suspect

No 1:1 mapping pattern found. The reconstruction has 11 system-level bullets and then many plan-ordered feature rationales under plan section headings. It does not enumerate S1-S9 or F1-F40, does not use gold order, and includes plan-specific operational items that are not gold targets.

## Plan-Derivation Spot Check

1. Reconstruction: "Presence must be qualified by all three signals: visible, focused, and recent pointer or key activity."
   PLAN support: §5 states the client qualifies presence only while all three conditions hold.

2. Reconstruction: "The server is the sole writer of bird personality, mood, perch/behavior schedule, weather, notebook, and age-based adoption availability."
   PLAN support: §2 uses the same server-writer rule and lists those state classes.

3. Reconstruction: "Reduced motion is a designed second renderer chosen by `prefers-reduced-motion` or an override."
   PLAN support: §6 states reduced motion is a designed second renderer and preserves the same canonical state/actions.

All three articulate sentences are directly grounded in PLAN passages.

## Verdict: PASS

The reconstruction reads as plan-derived. It uses the plan's headings, vocabulary, and ordering; it contains no gold IDs or rubric vocabulary; and spot-checked rationales are supported by the plan. No significant contamination signs were found.
