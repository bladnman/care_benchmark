# VALIDITY_AUDIT — CARE run 001

## ID leakage

Verdict: no ID leakage found. Searches for gold-style identifiers (`S1`-`S9`, `F1`-`F40`, `R-F`, `canonical #`) found no gold IDs in the frozen reconstruction. The word `canonical` appears, but it also appears in PLAN.md as ordinary architecture vocabulary for the server-authoritative state and is not a gold-list identifier.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "observational, unhurried relationship" | PLAN.md:6 exact phrase | Plan-derived |
| "server-side simulation tick" | PLAN.md:9,129,451 | Plan-derived |
| "first frame rendered has motion already underway" | PLAN.md:9 exact phrase | Plan-derived |
| "subtle bird behavioral reactions" | PLAN.md:10 exact phrase | Plan-derived |
| "naturalist field-notebook register" | PLAN.md:11 exact phrase | Plan-derived |
| "clear, standard matter-of-fact English without faux-warmth" | PLAN.md:13 exact phrase | Plan-derived |
| "quiet deepening of the user's relationship" | PLAN.md:813 exact phrase | Plan-derived |
| "Single Writer Principle" | PLAN.md:135,554 exact phrase | Plan-derived |
| "birdwatching is an idle activity" | PLAN.md:842 exact phrase | Plan-derived |
| "architectural constraints, not add-ons" | No exact phrase, but PLAN.md:37-42,598-602,733-755 make accessibility/privacy core scope and implementation | Benign synthesis, not rubric/gold jargon |

No rubric-side vocabulary such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `intent fidelity`, or `gold why` appears in the reconstruction.

## Heading Mirror

RECONSTRUCTION.md headings mirror the candidate plan's implementation sections rather than the gold-list taxonomy:

| Reconstruction heading | Closest PLAN heading | Gold-list mirror? |
|---|---|---|
| `## System-level intent` | Reconstruction-required section | No; required output shape |
| `## Per-feature whys` | Reconstruction-required section | No; required output shape |
| `### Executive Summary & Product Scope` | PLAN.md section 1 | No |
| `### System Architecture & Topology` | PLAN.md section 2 | No |
| `### Data Model & Database Schema` | PLAN.md section 3 | No |
| `### API Surface & Contract Specifications` | PLAN.md section 4 | No |
| `### Simulation Engine Design` | PLAN.md section 5 | No |
| `### Multi-Device Sync & Concurrency Model` | PLAN.md section 6 | No |
| `### Frontend Rendering & Micro-Motion Pipeline` | PLAN.md section 7 | No |
| `### Audio Pipeline & Procedural Call Synthesis` | PLAN.md section 8 | No |
| `### Accessibility Surfaces` | PLAN.md section 9 | No |
| `### Performance Budgets, Verification & Telemetry` | PLAN.md section 10 | No |
| `### Rollout & Phased Deployment Strategy` | PLAN.md section 11 | No |
| `### Risk Matrix & Technical Mitigations` | PLAN.md section 12 | No |
| `### Defensible Implementation Decisions` | PLAN.md section 13 | No |

No heading closely mirrors `GOLD_WHYS.md` section titles such as `System-level whys`, `Feature-level whys`, or canonical F/S identifiers.

## 1:1 Mapping Suspect

No 1:1 gold mapping is present. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold order, and contains many implementation bullets from PLAN.md that are outside the 40 scored feature whys. Its section order follows PLAN.md exactly. Several gold targets receive no neat corresponding item or are explicitly marked `NOT RECOVERABLE FROM PLAN`, which is the opposite of a suspiciously tidy gold-list mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan names this as a fundamental design principle. It appears in the server-side simulation tick, the mandate that 'the first frame rendered has motion already underway,' the 'Already Alive' first frame rules, and procedurally varied calls and greetings so 'no two moments sound identical.'" | PLAN.md:8-10 names principles; PLAN.md:610-615 contains the Already Alive first-frame rules. | Supported |
| "The plan says cooldowns 'prevent mechanic spamming' and 'forecloses any possibility of click-spamming while allowing legitimate experimentation.'" | PLAN.md:29 mentions offer cooldowns; PLAN.md:844 uses the quoted cooldown rationale. | Supported |
| "The rationale is to make 7-day changes detectable by regression tests but 'imperceptible to casual visual inspection,' with 21-day changes substantively shifting behavior." | PLAN.md:517-520 gives 7-day and 21-day calibration targets with the quoted wording. | Supported |

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no gold ID leakage, no rubric vocabulary, no heading mirror of the gold list, and no tidy S/F mapping. A small amount of synthesized language appears, but it is grounded in PLAN.md structure and does not look like contamination.
