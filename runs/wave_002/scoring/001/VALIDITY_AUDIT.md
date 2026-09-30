# VALIDITY_AUDIT - run 001

## ID Leakage

Verdict: no leakage found. A mechanical search for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` in the frozen reconstruction returned no hits. The same search was checked against the assigned PLAN, and there were no suspicious gold IDs to cross-reference.

## Vocabulary Check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing`.

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN-derived support |
|---|---|
| "durable bird identity is a release gate" | PLAN §1 makes UUID, vector, and call identity release gates. |
| "canonical server authority protects continuity" | PLAN §§1,3,7 assign canonical state to server tick and forbid client ownership. |
| "presence is intentionally narrow, honest, and non-invasive" | PLAN §6 gives the visible/focus/input formula and calls it an honest bounded protocol. |
| "absence is non-punitive" | PLAN §§1,7 forbid negative deltas, suffering, chores, and distrust. |
| "privacy boundaries are structural" | PLAN §§1,3,15 enforce database/telemetry separation and serializers. |
| "accessibility ships with the normal scene" | PLAN §§1,13,17 make accessibility release-blocking and first-class. |
| "calibration and release decisions avoid real-owner behavioral analytics" | PLAN §§7,16,18 reject actual-user drift averages and engagement funnels. |

## Heading Mirror

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Delivery contract and ambiguity decisions`
- `### Service architecture and data ownership`
- `### HTTP contracts and authorization`
- `### Presence, simulation, and recovery`
- `### Sync, interactions, adoption, and scene rendering`
- `### Calls, notebook, accessibility, visits, lifecycle, and rollout`

The first two headings are required by the phase-2A reconstruction prompt, not by the held-out gold list. The subsection headings mirror the PLAN's broad section groupings, not the gold-list S1-S9/F1-F40 taxonomy. No near-exact match to gold section headings was found beyond generic product-area words.

## 1:1 Mapping Suspect

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use canonical why IDs, and does not follow the gold list order. It groups material under the PLAN's architecture and delivery sections. It also marks several items `NOT RECOVERABLE FROM PLAN`, which is consistent with blind derivation rather than a neat gold-list mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support |
|---|---|
| "Presence is intentionally narrow, honest, and non-invasive." | PLAN §6 defines `qualified = visible && document.hasFocus() && trustedPointerOrKeyAge < 240s && !settled && ownerViewActive` and says this is an honest client protocol with bounds. |
| "Privacy boundaries are structural." | PLAN §1 says analytics cannot query the owner's simulation domain; PLAN §15 uses an allowlist collector and forbids behavioral fields. |
| "Reduced motion, narration, captioning, keyboard operation, and AA text contrast ship with the normal scene." | PLAN §1 names those surfaces as shipping with the normal scene and preserving affective quality; PLAN §13 makes failures release-blocking. |

## Verdict

PASS. The reconstruction reads as derived from the assigned PLAN: it uses PLAN headings and vocabulary, contains no gold IDs or rubric terms, does not map 1:1 to the gold why list, and its most articulate claims are directly supportable from PLAN passages.
