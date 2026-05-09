# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no leakage found. `RECONSTRUCTION.md` does not use gold IDs such as `S1`-`S9`, `F1`-`F40`, or `R-F01`. The audit search only matched ordinary plan vocabulary such as `canonical`; `PLAN.md` also uses `canonical` for server state, so this is not leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Server-authoritative, canonical state" | PLAN repeats canonical server state | plan-derived |
| "no client-side state to merge" | PLAN exact phrase | plan-derived |
| "monotonic upward (toward expressive)" | PLAN exact phrase | plan-derived |
| "not more wary, not less colorful" | PLAN exact phrase | plan-derived |
| "naturalist prose" | PLAN repeated phrase | plan-derived |
| "no spinner, no fade-in" | PLAN exact phrase | plan-derived |
| "different renderer for the same state" | PLAN exact phrase | plan-derived |
| "architectural rule enforced at the network/IAM level" | PLAN exact idea and wording | plan-derived |
| "specific email address" | PLAN visit invitation wording | plan-derived |

No rubric-side phrases such as `intent fidelity`, `multi-layer recovery`, `weight-3`, or gold IDs appear in the reconstruction.

## Heading Mirror Check

`RECONSTRUCTION.md` headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope`
- `### 2. Architecture, data model, and API surface`
- `### 5. Simulation engine design and sync model`
- `### 7. Frontend rendering pipeline, audio pipeline, and accessibility surfaces`
- `### 10. Performance budgets, observability, rollout, and risk controls`

These mirror PLAN section groups, not gold-list headings. They do not mirror `S1`-`S9`, `F1`-`F40`, or the gold file groups in a suspicious way.

## 1:1 Mapping Suspect Check

No suspicious 1:1 mapping. The reconstruction is organized around plan sections and plan bullets. It does not enumerate exactly 9 system whys plus 40 feature whys, and it includes many non-gold plan items such as service topology, magic-link compare-and-swap, browser support, bundle-size PR comments, and WebAudio smoke tests. It also marks several items `NOT RECOVERABLE FROM PLAN`, which is consistent with a plan-derived reconstruction rather than gold-list mirroring.

## Plan-Derivation Spot Check

1. Reconstruction: "The sync model says conflict is 'prevented (not resolved)' because there is 'no client-side state to merge.'" PLAN support: Section 6 has the same heading and phrase. Assessment: plan-derived.

2. Reconstruction: "The reduced-motion rendering path shares all the same data ... making it 'a different renderer for the same state.'" PLAN support: Section 7 says the reduced-motion path shares snapshot, mood, and perch-zone data and is "a different renderer for the same state." Assessment: plan-derived.

3. Reconstruction: "The analytics warehouse has 'no read access' to Aviary DB as an 'architectural rule enforced at the network/IAM level.'" PLAN support: Section 10 says the analytics warehouse has no read access to Aviary DB and this is enforced at the network/IAM level. Assessment: plan-derived.

## Verdict: PASS

The reconstruction reads as plan-derived. There is no gold ID leakage, no rubric vocabulary leakage, no gold-order 1:1 mapping, and the most articulate reconstruction sentences are directly grounded in the plan.
