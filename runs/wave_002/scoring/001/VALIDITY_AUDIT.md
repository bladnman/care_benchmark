# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict for this check: PASS.

Mechanical search of the frozen reconstruction found no gold identifiers such as `F1`, `F40`, `S1`, `S9`, `R-F01`, or `R-S01`. The reconstruction does not use the scorer-side taxonomy or external why IDs.

## Vocabulary Check

Verdict for this check: PASS.

Rubric-side terms such as `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, and `intent fidelity` do not appear. The only superficially concerning token was `golden replay fixtures`, but that phrase is plan-derived: PLAN section 6 says `Golden replay fixtures must yield identical state on restart or worker reassignment.`

Sampled reconstruction phrases and grounding:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `Absence has no moral weight` | PLAN says `a missing settle gesture and a long absence have no penalty` and `absence causes no harm` | Plan-derived paraphrase |
| `Privacy is structural` | PLAN requires analytics separation at network/credential layers and no account/bird fields in metrics | Plan-derived paraphrase |
| `not a later patch` | PLAN says equal access/account controls are `launch requirements, not a later patch` | Directly grounded |
| `software startup` | PLAN risk row names `First frame feels like software starting` | Directly grounded |
| `deliberate grant` | PLAN invitation records use no global switch and each invite is deliberate | Plan-derived/direct |
| `quiet visit log` | PLAN says host sees a quiet visit log with no badge | Directly grounded |

## Heading Mirror

Verdict for this check: PASS.

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Delivery contract`
- `### System boundaries and deployment shape`
- `### Persistent data and invariants`
- `### API contracts and command handling`
- `### Presence, concurrency, and synchronization`
- `### Simulation engine and content generation`
- `### Browser scene, audio, and accessibility`
- `### Budgets, instrumentation, and gates`
- `### Build sequence and rollout`
- `### Risk register and release definition`

The two top-level headings are required by the phase-2A prompt. The subsection headings mirror the PLAN's own section structure, not the gold-list sections or S/F why list.

## 1:1 Mapping Suspect

Verdict for this check: PASS.

The reconstruction is not a neat S1-S9 / F1-F40 mapping. It follows PLAN sections, includes many implementation features that are not gold-why-bearing, and marks several exact details `NOT RECOVERABLE FROM PLAN`. The order is plan-derived rather than gold-derived.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| `The first impression should feel like ongoing life, not software startup.` | PLAN requires no spinner/static entry, first bird without extra round trip, phase-anchored motion, and lists `First frame feels like software starting` as a risk. | Grounded |
| `Privacy is structural.` | PLAN restricts analytics credentials/network access, emits no account/bird/event/vector data, and destroys per-account encryption keys on deletion. | Grounded |
| `Visits are deliberate, revocable, read-only glimpses.` | PLAN has named invites, no global sharing switch, per-pull revocation, scoped read-only sessions, and no visitor event ingestion. | Grounded |

## Verdict

PASS. The frozen reconstruction reads as a plan-derived reconstruction with no gold-ID leakage, no scorer/rubric vocabulary, no heading mirror to the held-out gold list, and no suspicious 1:1 target mapping. Some phrases are inferential rather than verbatim, but the spot checks tie them back to the PLAN.
