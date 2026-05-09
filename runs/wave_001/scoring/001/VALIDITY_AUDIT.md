# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as plan-derived. I found no gold-ID leakage, no rubric vocabulary, no 1:1 S/F mapping to the gold list, and the strongest reconstruction claims are directly supported by PLAN passages.

## 1. ID Leakage Check

No leakage found.

- Search in `RECONSTRUCTION.md` for gold IDs and rubric-like identifiers (`S1`-`S9`, `F1`-`F40`, `R-F*`) returned no hits.
- The reconstruction uses plain plan-derived headings and bullets rather than gold taxonomy.
- Because there were no reconstruction hits, no unmatched ID cross-reference against PLAN was needed.

## 2. Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "affective constraints are engineering constraints" | PLAN introduction: "affective constraints from the PRD as engineering constraints" | Plan-derived |
| "last-write-wins on personality silently destroys drift" | PLAN Appendix A: "last-write-wins on personality silently destroys drift" | Exact plan phrase |
| "canned audio kills the spell" | PLAN Appendix A: "canned audio kills the spell" | Exact plan phrase |
| "felt-coherence" | PLAN §2.4 uses failure modes that "hurt felt-coherence" | Plan-derived |
| "not a fallback" | PLAN §7.4: reduced-motion is "not a fallback" | Exact plan phrase |
| "not a load state" | PLAN §7.1: "The static first frame is not a load state" | Exact plan phrase |
| "observes the aviary, never the user" | PLAN §5.6: "The notebook makes observations of the aviary, never of the user" | Exact plan phrase |
| "operational health" vs "user behavior aggregation" | PLAN §10.6: "operational health (allowed) vs. user behavior aggregation (not allowed)" | Exact plan phrase |

No sampled phrase looked like rubric-side scoring vocabulary. Terms such as `multi-layer`, `feature-level fidelity`, `weight-3`, `intent fidelity`, and `load-bearing` do not appear in the reconstruction.

## 3. Heading Mirror Check

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Architecture`
- `### Data Model`
- `### API Surface`
- `### Simulation Engine`
- `### Sync Model`
- `### Frontend Rendering Pipeline`
- `### Audio Pipeline`
- `### Accessibility`
- `### Performance and Observability`
- `### Rollout, Flags, and Migration`

These mirror the PLAN's implementation sections, not GOLD_WHYS section titles. The two top headings are the expected reconstruction format rather than gold-list leakage. No heading closely mirrors a gold why title like `feels-alive-not-robotic` or `presence-definition`.

## 4. 1:1 Mapping Suspect Check

No 1:1 mapping found.

- The system section has 11 numbered principles, not the gold list's 9 system whys.
- The per-feature section is grouped by plan architecture sections and contains many more than 40 bullets.
- It does not march through S1-S9 or F1-F40 in gold order.
- The structure is consistent with a reconstructor expanding the assigned PLAN, not with a scorer-facing gold ledger.

## 5. Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Judgment |
|---|---|---|
| "The central boundary is: The server is the only writer of canonical aviary state." | PLAN §2.2 states the same boundary and explains clients render snapshots and submit append-only events. | Supported |
| "The static frame is not a load state; it is the aviary." | PLAN §7.1 says exactly that and describes inline first-frame rendering. | Supported |
| "The feature is self-limiting by design: opt-in invitations, revocable links, no co-presence, no chat, no avatars, and no default notifications prevent social-network creep." | PLAN §§1.1, 1.2, 4.3, and 12.12 list opt-in visits, revocation, read-only view, no chat/avatars, no default notifications, and social-network creep risk. | Supported |

## 6. Final Verdict

**PASS.** The reconstruction contains a few plan-compressed or overgeneralized rationales, but those are scoring misses rather than contamination signatures. Its vocabulary, headings, ordering, and strongest claims are grounded in the PLAN and do not reveal gold-side IDs or rubric machinery.
