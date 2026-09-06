# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A targeted search of the frozen reconstruction found no gold IDs such as `S1`-`S9`, `F1`-`F40`, `R-F01`, or rubric labels such as `intent fidelity`, `feature-level fidelity`, `weight-3`, `multi-layer recovery`, or `load-bearing`. The reconstruction uses numbered headings like `### 1. Scope & System Boundaries`, but those mirror the PLAN section numbers, not gold IDs.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "ambient, unhurried presence as the central relationship model" | PLAN.md:5 and PLAN.md:12 describe the ambient aviary and unhurried presence. | Derived; "central relationship model" is a summarizing phrase, not rubric vocabulary. |
| "Slow, bounded, non-punitive change" | PLAN.md:28 and PLAN.md:340 ground non-punitive monotonic drift. | Plan-derived. |
| "single source of canonical reality" | PLAN.md:36 and PLAN.md:377-380 ground server-owned canonical state. | Plan-derived paraphrase. |
| "Privacy is structural, not cosmetic" | PLAN.md:91-93 and PLAN.md:562-564 ground structural telemetry isolation. | Plan-derived paraphrase. |
| "Notice, never announce" | PLAN.md:601 uses the same phrase; PLAN.md:13 and PLAN.md:30 ground the rule. | Directly plan-derived. |
| "primary aesthetic surface rather than an afterthought" | PLAN.md:515 uses this wording. | Directly plan-derived. |
| "Strict Architectural Wall" | PLAN.md:562 uses this heading. | Directly plan-derived. |
| "product integrity" | PLAN.md:509 uses this phrase for WebAudio fallback. | Directly plan-derived. |

No suspicious rubric-side vocabulary appears freely in the reconstruction.

## Heading Mirror

Reconstruction headings are:

| Reconstruction heading | Closest source | Assessment |
|---|---|---|
| `## System-level intent` | Phase-2A expected output shape, not a gold-list title. | Not suspicious. |
| `## Per-feature whys` | Phase-2A expected output shape, not a gold-list title. | Not suspicious. |
| `### 1. Scope & System Boundaries` through `### 12. Technical Risks & Mitigation Strategies` | PLAN.md section headings. | Mirrors PLAN, not GOLD_WHYS. |

The headings do not echo gold-list titles like `System-level whys`, `Feature-level whys`, or F/S IDs.

## 1:1 Mapping Suspect

No 1:1 gold mapping found. The reconstruction does not enumerate S1-S9 or F1-F40 in order. Instead, it has twelve system-intent bullets and then many plan-section-derived bullets, including numerous features that are not in the gold-why list. This is consistent with reading PLAN.md, whose sections are numbered 1-12.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The server is the single source of canonical reality." Supporting PLAN passage: PLAN.md:36 states that the server owns all state, simulation, drift calculations, and persistence; PLAN.md:377-380 defines the canonical single-writer model. Verdict: grounded.

2. Reconstruction sentence: "The rationale is to make birds evolve from actual human attention rather than passive open tabs." Supporting PLAN passage: PLAN.md:382-388 requires visible document, focus, recent input, and deduped heartbeats; PLAN.md:600 describes the forgotten-window false-presence risk. Verdict: grounded.

3. Reconstruction sentence: "Allowed telemetry is operational aggregate health, while a Strict Architectural Wall keeps user state, interaction details, and notebook content out of the analytics sink." Supporting PLAN passage: PLAN.md:558-564 allows operational aggregates and excludes account IDs, bird IDs, personality traits, interaction event details, and notebook content. Verdict: grounded.

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It mirrors PLAN.md structure, contains no gold IDs or rubric-specific scoring language, and its most articulate claims can be grounded in allowed PLAN passages. Minor paraphrases such as "central relationship model" are inferential summaries, not contamination signals.
