# VALIDITY_AUDIT - CARE run 001

## ID leakage

Mechanical search found no gold IDs or rebuild IDs in `RECONSTRUCTION.md`: no `F1`-style feature IDs, no `S1`-style system IDs, and no `R-F`/`R-S` tokens. The same search found only one unrelated plan-side word, `golden`, in `PLAN.md` (`immutable golden histories`), not in the reconstruction. No ID leakage.

## Vocabulary check

Sampled reconstruction phrases and plan grounding:

| Reconstruction phrase | Appears in / derives from PLAN? | Finding |
|---|---|---|
| `primary quality bar is continuity` | Exact phrase in PLAN §1. | Clean. |
| `first frame already in progress` | PLAN §1 and §8.1 repeatedly require this. | Clean. |
| `same birds persist across devices` | PLAN §1 and identity/sync sections ground this. | Clean. |
| `one canonical, server-authored aviary` | Exact plan rule in §1. | Clean. |
| `Presence is not a page view` | Paraphrase of PLAN §1/§6.2 open-tab rule. | Clean. |
| `procedural, specific, and non-canned` | Derives from procedural calls and specific naturalist surfaces in PLAN. | Clean. |
| `Accessibility is part of v1 value` | PLAN says accessibility ships with v1 and cannot trail beta. | Clean. |
| `host relationship being reshaped or exposed` | PLAN risk table uses this framing for visitor risk. | Clean. |
| `severity-one data-loss event` | Exact plan wording for vector reset. | Clean. |
| `is the service healthy?` | Exact dashboard framing in PLAN §12.2. | Clean. |

No scorer-side vocabulary such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity` appears in the frozen reconstruction.

## Heading mirror

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Product contract and implementation decisions`
- `### V1 scope and acceptance boundary`
- `### System architecture, persistence model, and API/client contracts`
- `### Simulation engine`
- `### Multi-device synchronization and failure behavior`
- `### Frontend rendering and interaction pipeline`
- `### Procedural audio pipeline`
- `### Accessibility implementation`
- `### Security, privacy, and lifecycle`
- `### Performance budgets and observability`
- `### Verification strategy`
- `### Delivery sequence and rollout`

The first two headings match the required reconstruction output format. The remaining headings mirror the PLAN's section structure, sometimes compressed across adjacent plan sections. They do not mirror `GOLD_WHYS.md` system titles or feature-list groupings.

## 1:1 mapping suspect

No 1:1 mapping to S1-S9/F1-F40 is visible. The reconstruction does not use gold IDs, does not enumerate 49 target whys, and does not proceed in gold-list order. It follows the PLAN's implementation sections and includes many plan-specific items that are not gold targets, such as timezone ambiguity resolution, edge BFF, IndexedDB outbox, rollout phases, and kill switches. Not suspect.

## Plan-derivation spot check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| `The opening contract says the "primary quality bar is continuity": first frame "already in progress," the "same birds persist across devices," and attention changes them "slowly without making absence harmful."` | PLAN §1 contains those phrases in the first paragraph. | Grounded. |
| `A bird pins a profile version so later asset releases cannot silently replace its identity; IDs are never reused or changed by rename, migration, sync, or export/import work.` | PLAN §4.1 defines pinned species profiles; §4.3 says bird IDs are never reused/regenerated/changed by rename, migration, sync, or export/import. | Grounded. |
| `Accessibility acceptance is affective as well as mechanical: a screen-reader or reduced-motion session must contain continuing, specific aviary behavior and not degrade into a state list or static placeholder.` | PLAN §10 says this nearly verbatim. | Grounded. |

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It contains no gold ID leakage, no scorer vocabulary, no gold-heading mirror, and no neat 1:1 target mapping. Its strongest phrases can be traced directly to the PLAN, and its omissions look like normal compression rather than contamination.
