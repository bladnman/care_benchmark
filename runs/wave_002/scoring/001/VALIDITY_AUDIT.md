# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: no ID leakage found. I searched the frozen reconstruction for gold IDs and rubric-style identifiers such as S1-S9, F1-F40, R-Fxx, weight-3, multi-layer, feature-level fidelity, intent fidelity, gold, rubric, and load-bearing. There were no hits in RECONSTRUCTION.md, and no corresponding suspicious ID terms in PLAN.md.

## Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Calm, non-gamified, non-punitive ambient care" | PLAN non-goals: no achievements/streaks/scores; no dying/hunger/negative drift | Plan-derived synthesis |
| "Server-authoritative, append-only truth" | PLAN: "server is the authoritative source of truth" and "append-only interaction events" | Plan-derived synthesis |
| "Slow time, with visible but subtle change" | PLAN: slow personality drift; visible over weeks but imperceptible over days | Plan-derived synthesis |
| "Naturalist prose as product voice" | PLAN: naturalist prose notebook and screen-reader narration | Plan-derived synthesis |
| "Privacy and bounded sociality" | PLAN: encrypted email, opt-in read-only visits, no public discovery/leaderboards, aggregate telemetry | Plan-derived synthesis |
| "Accessibility as a first-class surface" | PLAN: screen-reader narration, reduced motion, captions, WCAG AA, keyboard traversal | Plan-derived synthesis |
| "core affective experience" | PLAN uses "core affective experience" in rollout instrumentation | Exact plan wording |
| "phase-canceling artifacts of looped tracks" | PLAN audio pipeline uses this phrase | Exact plan wording |

No rubric-side phrases such as multi-layer recovery, feature-level fidelity, weight-3, load-bearing, or intent fidelity appear in the reconstruction.

## Heading mirror

The reconstruction headings mirror the PLAN structure, not GOLD_WHYS.md. Top headings are "System-level intent" and "Per-feature whys"; subheadings are Scope, Out of Scope and Non-goals, Architecture, Data model, API surface, Simulation engine design, Sync model, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Performance budgets and observability, and Rollout. These correspond to PLAN headings and do not mirror gold-list section titles or IDs.

## 1:1 mapping suspect

No 1:1 gold mapping is visible. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not follow the gold order. It instead walks the candidate PLAN sections and feature bullets, including many non-gold features and several "NOT RECOVERABLE FROM PLAN" entries. That pattern is consistent with plan-derived reconstruction.

## Plan-derivation spot check

1. Reconstruction: "The simulation consumes the append-only log, and last-write-wins is explicitly avoided to prevent device conflict data loss." PLAN support: "The engine consumes the client's append-only log" and "Last-write-wins is explicitly avoided to prevent device conflict data loss." Supported.

2. Reconstruction: "A slow 30-60s ARIA-live feed describes the aviary in naturalist prose, matching the slow aviary pace without overwhelming the queue." PLAN support: "A slow (30-60s) ARIA-live feed" and the accessibility risk about prose staying synced with the slow aviary pace without overwhelming the screen-reader queue. Supported.

3. Reconstruction: "Because clients are snapshot-readers and event-appenders, simultaneous laptop and phone use resolves to the same canonical state computed by the server." PLAN support: "all clients are just snapshot-readers and event-appenders" and simultaneous laptop/phone sees "the same canonical state computed by the server." Supported.

## Verdict

PASS. The reconstruction reads as a plan-derived synthesis: no gold IDs, no rubric vocabulary, no gold-order mapping, and the most articulate claims are directly grounded in PLAN wording. The few abstract headings are ordinary summaries of the plan rather than contamination signatures.
