# VALIDITY_AUDIT - CARE run 001

## Gold ID leakage check

Verdict: no leakage found. A narrow search of the frozen reconstruction for gold IDs and rubric-style identifiers (`S1`, `F1`, `F40`, `R-F...`) returned no hits. The same search over the plan also returned no such IDs, so there is no reconstruction-only ID leakage to report.

## Vocabulary check

Sampled load-bearing phrases in `RECONSTRUCTION.md` all appear to be plan-derived rather than gold/rubric-derived:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "server tick continuity" | PLAN.md:558 says "Server tick continuity" | plan-derived |
| "mid-action first frame" | PLAN.md:103 and PLAN.md:414 specify mid-action/mid-preen first paint | plan-derived |
| "procedural calls" | PLAN.md:17 and PLAN.md:330-336 describe procedural call grammar | plan-derived |
| "tick continues transitions while user away" | PLAN.md:328 says transitions persist while away | plan-derived |
| "Bird greeting only" | PLAN.md:559 says "Bird greeting only" | plan-derived |
| "no toasts/streaks" | PLAN.md:559 and PLAN.md:27 support this | plan-derived |
| "drift spine" | PLAN.md:563 says "Strict presence definition as drift spine" | plan-derived |
| "Simulation DB isolated" | PLAN.md:85 says simulation DB is isolated | plan-derived |
| "Own aesthetic, not 'broken static'" | PLAN.md:410 uses the same phrase | plan-derived |
| "observation voice not ARIA state dumps" | PLAN.md:435 uses the same phrase | plan-derived |

No rubric-side vocabulary such as `multi-layer`, `feature-level fidelity`, `weight-3`, `gold why`, or `intent fidelity` appeared in the reconstruction search.

## Heading Mirror Check

Reconstruction headings:

| Heading | Comparison to gold/rubric headings | Finding |
|---|---|---|
| `## System-level intent` | Similar to the phase-2A required output section, not the gold title `System-level whys` | acceptable |
| `## Per-feature whys` | Similar to the phase-2A required output section, not a 1:1 gold list title | acceptable |
| `### Scope and v1 commitments` | Mirrors PLAN scope, not gold sections | acceptable |
| `### Ambiguity calls` | Mirrors PLAN section | acceptable |
| `### Architecture, data, and API` | Mirrors PLAN content clusters | acceptable |
| `### Simulation engine design` | Mirrors PLAN section | acceptable |
| `### Sync, frontend, audio, accessibility, performance, and rollout` | Mirrors PLAN implementation sections | acceptable |

The headings do not mirror the gold-list headings closely enough to suggest contamination.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not present exactly 49 target rows. Its system section has 11 bullets, including plan-specific additions such as performance discipline and limited sociality. Its feature section is grouped by plan sections and includes many non-gold operational items. The overlap with gold targets is expected because the PLAN itself has a traceability table of principles and broad implementation sections.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The product should let birds respond without system messages announcing the user's behavior back to them."
   - PLAN support: PLAN.md:347 says no toast/welcome/days-gone text; PLAN.md:559 says "Bird greeting only"; PLAN.md:521 flags welcome-toast regression.
   - Assessment: supported.

2. Reconstruction sentence: "Visitor tokens have 'events API 403' and 'visitor attention cannot drift host birds.'"
   - PLAN support: PLAN.md:249 gives the visitor snapshot path; PLAN.md:251 says no offer/listen-in/settle/write notebook; PLAN.md:379 says visitor attention cannot drift host birds.
   - Assessment: supported.

3. Reconstruction sentence: "Reduced-motion mode is inclusion with charm: 'Own aesthetic, not broken static,' while keeping color phase, calls, and captions."
   - PLAN support: PLAN.md:410 has the same reduced-motion phrasing; PLAN.md:443 requires a11y to ship with v1.
   - Assessment: supported.

## Verdict

PASS. The reconstruction reads as plan-derived: it borrows many phrases directly from the PLAN, has no gold-ID leakage, no rubric vocabulary hits, no 1:1 mapping to the gold list, and the checked articulate sentences are supported by PLAN passages. The main scoring losses are ordinary compression and rule-without-why omissions, not evidence of contamination.
