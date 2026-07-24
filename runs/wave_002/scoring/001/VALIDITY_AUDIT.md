# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage

Verdict: PASS.

Mechanical grep over the frozen reconstruction found no gold IDs or scorer IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. The reconstruction does not use external gold taxonomy.

## Vocabulary Check

Verdict: PASS.

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled phrases and plan derivation:

| Reconstruction phrase | Status | PLAN support |
|---|---|---|
| "Server-authored life, client-rendered presence" | Derived synthesis | PLAN says the web client "Owns no canonical state" and server owns personality, moods, canonical positions, and drift. |
| "Privacy boundary by architecture, not just policy" | Derived synthesis | PLAN says the simulation DB is never read by analytics pipelines and the boundary is enforced at infra level. |
| "Ambient, non-gamified, non-social product shape" | Derived synthesis | PLAN explicitly excludes gamification, Tamagotchi mechanics, social-network surfaces, and notifications. |
| "Slow, monotonic expressiveness instead of obligation or decay" | Derived synthesis | PLAN specifies monotonic drift and absence as zero, never negative, delta. |
| "Naturalist, observation-voiced prose" | Grounded | PLAN uses naturalist narration/captions/notebook prose and forbids user-behavior notebook phrasing. |
| "Accessibility is part of v1, not a fallback" | Grounded | PLAN says accessibility surfaces ship with v1 and are launch-blocking, not v1.1. |
| "Immediate, quiet, continuously framed experience" | Derived synthesis | PLAN says no spinner, no entry animation, quiet field loading, first bird under 500ms, and birds mid-action. |

The synthesized labels do not appear to come from the gold list; they compress plan language in the reconstructor's own grouping.

## Heading Mirror

Verdict: PASS.

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope`
- `### 2. Architecture`
- `### 3. Data model`
- `### 4. API surface`
- `### 5. Simulation engine design`
- `### 6. Sync model`
- `### 7. Frontend rendering pipeline`
- `### 8. Audio pipeline`
- `### 9. Accessibility surfaces`
- `### 10. Performance budgets and observability`
- `### 11. Rollout`
- `### 12. Risks`

These mirror PLAN.md section headings, not GOLD_WHYS.md headings. They do not echo the gold S/F taxonomy.

## 1:1 Mapping Suspect

Verdict: PASS.

The reconstruction does not provide a neat S1-S9 or F1-F40 list, does not use gold IDs, and does not follow the gold feature order. Instead it follows PLAN.md's 12 sections and reconstructs many implementation rows that are not in the 40 gold-why list. Several gold whys are missing or marked NOT RECOVERABLE FROM PLAN, which also argues against a gold-list mirror.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Result |
|---|---|---|
| "The simulation DB is never read by analytics pipelines." | PLAN sec. 2 says the simulation DB is never read by analytics pipelines, with no warehouse connection and separate credentials. | Grounded. |
| "The risk section says any shortcut such as 'tab open' would corrupt drift population-wide." | PLAN sec. 12 says any shortcut like tab open corrupts drift population-wide, silently. | Grounded. |
| "Reduced motion is 'a designed surface, not a stripped fallback'." | PLAN sec. 7 uses that exact rationale for reduced-motion mode. | Grounded. |
| "The audio risk says repetitive-sounding procedural calls break the spell as badly as loops." | PLAN sec. 12 says repetitive-sounding procedural calls break the spell as badly as loops. | Grounded. |

## Verdict: PASS

No significant contamination signatures were found. The reconstruction is broad and articulate, but it is structured around the candidate PLAN and uses plan-derived vocabulary. It lacks gold IDs, scorer-side vocabulary, and a 1:1 gold mapping; the missing and NOT RECOVERABLE entries look like honest plan-derived reconstruction rather than leakage.
