# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS.

I searched the frozen reconstruction for gold IDs and rubric-like identifiers (`S1`, `F1`-style IDs, `R-F`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `gold`, `rubric`, and `load-bearing`). No hits appeared in RECONSTRUCTION.md or in PLAN.md for those suspect terms. The reconstruction does not use the gold-list numbering or a hidden scoring taxonomy.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "believable relationship over weeks" | PLAN section 1 uses the same phrase. | Plan-derived. |
| "Identity and personality survive renames, device changes, migrations, and absence" | PLAN section 1 uses the same sentence. | Plan-derived. |
| "ordinary tab closure is a complete and valid goodbye" | PLAN section 1 says ordinary tab closure is valid. | Plan-derived. |
| "one human minute is at most one minute of presence" | PLAN section 5.2 says this directly. | Plan-derived. |
| "telemetry firewall" | PLAN sections 12 and 10.3 describe telemetry firewall/analytics barriers. | Plan-derived. |
| "same quiet aliveness is available through sound, captions, narration, or reduced motion" | PLAN final paragraph uses this formulation. | Plan-derived. |
| "relationship-first intent" | Not exact PLAN vocabulary, but directly summarizes PLAN's "believable relationship over weeks" language. | Benign abstraction. |
| "Naturalism should avoid spectacle" | Not exact wording; supported by PLAN's quiet sky, no dramatic weather, no entry sequence, and no attention-seeking event language. | Benign abstraction. |

I found no rubric-side vocabulary used freely. The reconstruction reads as a synthesis of PLAN language, not as gold-list leakage.

## Heading Mirror Check

RECONSTRUCTION headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### Product contract and scope`
- `### Decisions where details are open or conflicting`
- `### Architecture and ownership boundaries`
- `### API and command contracts`
- `### Canonical simulation engine`
- `### Snapshot sync and consistency`
- `### Frontend scene and rendering pipeline`
- `### Procedural audio and caption runtime`
- `### Accessibility and notebook surfaces`
- `### Identity, visiting, privacy, and account lifecycle`
- `### Performance, operations, build, and rollout`

These mirror PLAN section headings, not GOLD_WHYS headings. They do not echo the gold-list S1-S9 or F1-F40 section labels in order or wording.

## 1:1 Mapping Suspect Check

Verdict for this check: PASS.

The reconstruction does not provide a neat S1-S9 then F1-F40 list. Its per-feature section is organized by PLAN sections and contains many bullets that are outside the 49 scored whys. It includes honest `NOT RECOVERABLE FROM PLAN` entries for several non-gold implementation choices. This is not a suspicious 1:1 mapping to the gold list.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN grounding | Assessment |
|---|---|---|
| "Privacy is a design boundary, not just a policy." | PLAN sections 10.3 and 11.2 require a telemetry firewall, no request-body logs, no analytics over per-bird events, and code/schema/network egress checks. | Supported synthesis. |
| "Calibration must use synthetic histories and explicit studies, not production engagement mining." | PLAN sections 5.3 and 13 require synthetic schedules, consented qualitative studies, no production drift mining, and no A/B of bird personalities for engagement. | Supported. |
| "Accessibility is an equal way to experience the same aviary." | PLAN sections 1, 7.4, 9, and 12 require sound/captions/narration/reduced motion, authored still-pose cross-fades, and no post-launch accessibility deferral. | Supported synthesis. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses PLAN section headings, PLAN vocabulary, and PLAN-derived abstractions; it does not leak gold IDs, rubric vocabulary, or a 1:1 gold-list order. Minor synthesized phrases are supported by PLAN passages and do not look like external taxonomy.
