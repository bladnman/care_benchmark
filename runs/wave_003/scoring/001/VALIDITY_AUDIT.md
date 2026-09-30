# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: no leakage found.

Mechanical search of the frozen reconstruction found no tokens matching gold IDs such as `F1`, `S1`, `R-F01`, or `R-S01`. No external gold taxonomy appears in the reconstruction. The PLAN also does not contain such IDs in the searched pattern, so there are no reconstruction-only ID hits to report.

## Vocabulary check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled phrases read as plan-derived:

| Reconstruction phrase | PLAN support |
|---|---|
| "Real continuity, not arrival simulation" | PLAN sec 1 says the server advances birds while nobody is watching and the browser renders current lives. |
| "Server authority over the relationship" | PLAN sec 3 says clients never run the domain tick and only read projections. |
| "Honest owner presence, not engagement farming" | PLAN sec 6 defines the three-signal presence conjunction and rejects unattended windows. |
| "Accessibility carries the affective core" | PLAN sec 16 names the risk "Accessibility loses the affective core" and sec 1 requires accessibility at initial release. |
| "Canonical cross-device convergence" | PLAN sec 8 says same scene_revision yields identical shared-world projection bytes. |
| "Recovery preserves birds rather than recreating them" | PLAN secs 4, 12, and 16 reject reseeding and resetting aviaries. |
| "least intrusive notification channel" | PLAN sec 2 uses exactly that reasoning for optional visit notice inside settings. |
| "not frozen default silhouettes" | PLAN sec 9 says reduced-motion birds are not frozen default silhouettes. |

## Heading mirror

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Purpose, scope, and delivery rules`
- `### 2. Decisions where the specification leaves gaps`
- `### 3. Architecture and authority boundaries`
- `### 4. Data model, durability, and retention`
- `### 5. API contracts and authorization`
- `### 6. Precise presence, listen duration, and session lifecycle`
- `### 7. Server simulation engine`
- `### 8. Sync and consistency across devices`
- `### 9. Frontend scene and interaction rendering`
- `### 10. Procedural audio and call captions`
- `### 11. Field notebook and accessible prose`
- `### 12. Security, account lifecycle, and privacy enforcement`
- `### 13. Performance budgets and operational observability`
- `### 14. Verification and release acceptance`
- `### 15. Delivery sequence and rollout`
- `### 16. Main risks and mitigation owners`

These mirror the PLAN's own section structure, not the gold-list sections. They do not mirror the scorer-side S1-S9/F1-F40 taxonomy.

## 1:1 mapping suspect

Verdict: not suspect.

The reconstruction does not provide neat S1-S9 and F1-F40 items in gold order. It has 12 system-intent bullets and then a long section-by-section reconstruction aligned to the PLAN's 16 headings. Several gold whys are merged into broad plan sections, and some features are explicitly marked `NOT RECOVERABLE FROM PLAN`. This is the expected shape for a blind plan-derived reconstruction.

## Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage |
|---|---|
| "The filter retains a short positive tail from attention before the user left, consistent with absence-time drift from prior inputs." | PLAN sec 7 says the filter retains a short positive tail from attention before the user left and never lowers traits. |
| "Visitors can use their own sound permission, captions, reduced motion, and narration controls, but cannot focus a bird into listen-in, activate greetings, offer, settle, alter host settings, or browse the host's private notebook." | PLAN sec 5 contains the same visitor permission boundaries. |
| "A renderer that runs cleanly for one minute does not pass." | PLAN sec 13 uses this sentence in the memory/cleanup discipline section. |

All three sampled articulate claims are directly grounded in the PLAN.

## Verdict: PASS

The reconstruction reads as plan-derived. There is no gold ID leakage, no scorer/rubric vocabulary, no gold-list heading mirror, and no 1:1 mapping to the held-out S/F target list. Its structure tracks the PLAN's headings and vocabulary, with honest non-recovery markings where the plan did not make a why recoverable.
