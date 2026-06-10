# VALIDITY_AUDIT — CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as plan-derived. I found no gold-ID leakage, no rubric vocabulary leakage, no 1:1 mapping to S1-S9/F1-F40, and its headings mirror the candidate PLAN rather than GOLD_WHYS. The main artifact risk is normal reconstruction compression, not contamination.

## ID leakage

Search target: gold IDs and rubric-like IDs such as `S1`, `F1`, `R-F01`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `gold`, and `rubric`.

- **RECONSTRUCTION hits:** none.
- **PLAN cross-reference:** PLAN contains two uses of `load-bearing`; RECONSTRUCTION does not use that term. No suspicious ID or taxonomy appears in RECONSTRUCTION.
- **Assessment:** PASS.

## Vocabulary check

Sampled phrases from RECONSTRUCTION and supporting PLAN passages:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "The server simulates, the client performs" | PLAN intro says the same sentence governs every decision. | Plan-derived. |
| "one canonical continuously-advancing simulation" | PLAN call #2 describes hot ticking plus exact lazy catch-up and a canonical continuously advancing simulation. | Plan-derived. |
| "Sync is made structural, not conventional" | PLAN §7 says single-writer architecture makes sync a property, not a feature. | Plan-derived. |
| "Privacy is enforced by shape, not policy" | PLAN §§12-13 enforce metric schema, DB-role, and analytics boundaries. | Plan-derived paraphrase. |
| "Observe the aviary, not the user" | PLAN §6.5 says notebook inputs are aviary observations only and cannot reference user-behavior aggregates. | Plan-derived. |
| "Accessibility ships with the feature" | PLAN §10 says accessibility items are part of each feature definition of done and no separate accessibility milestone exists. | Plan-derived. |
| "Recognizability is a structural guarantee" | PLAN §§4 and 6.4 tie stable bird IDs, traits, and call_seed to continuity and recognizability. | Plan-derived. |
| "co-presence and visitor interactivity are unrepresentable" | PLAN §5 says visitor router has no event endpoint and visitor interactivity is unrepresentable. | Plan-derived. |
| "no banner system exists" | PLAN §14 says no banner system exists to abuse. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | This is a reconstruction-side self-assessment style, not gold/rubric taxonomy; it appears attached to plan features, not gold IDs. | Not suspicious. |

No sampled phrase looked imported from the rubric or gold list without plan support.

## Heading mirror

RECONSTRUCTION headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope`
- `### Adjudicated ambiguities and interpretation decisions`
- `### Architecture`
- `### Data model`
- `### API surface`
- `### Simulation engine design`
- `### Sync model`
- `### Frontend rendering pipeline`
- `### Audio pipeline`
- `### Accessibility surfaces`
- `### The voice system`
- `### Performance budgets and observability`
- `### Security and privacy engineering`
- `### Rollout`
- `### Tuning-constant register`

Comparison:

- These headings closely mirror PLAN sections 1-16, with numbering removed.
- They do **not** mirror GOLD_WHYS headings such as `S1 — feels-alive-not-robotic`, `F1 — presence-definition`, or file-grouped gold sections.
- The required `System-level intent` / `Per-feature whys` wrapper comes from the reconstruction task format, not from gold leakage.

Assessment: PASS.

## 1:1 mapping suspect

The reconstruction does not provide neat corresponding items for every S1-S9 and F1-F40 in gold order. Instead, it follows the PLAN structure: scope, calls, architecture, data model, API, engine, sync, frontend, audio, accessibility, voice, performance, security, rollout, tuning. It includes many plan-feature bullets and several NOT RECOVERABLE statements, but these are not aligned to gold IDs or the gold order.

Assessment: PASS.

## Plan-derivation spot check

1. RECONSTRUCTION: "The telemetry registry rejects any metric with a per-account or per-bird dimension."
   PLAN support: §12 says the metric schema registry has no account/bird dimension type and adding one is a build failure.
   Result: grounded.

2. RECONSTRUCTION: "The visitor endpoint set is separate, read-only, has no notebook, no greeting directive, and no event acceptance."
   PLAN support: §5 visitor routes specify read-only snapshot, no greeting directive, no notebook, no event acceptance.
   Result: grounded.

3. RECONSTRUCTION: "Mood decays toward time-of-day baseline instead of hard-resetting, so tab-open and overnight catch-up do not snap state."
   PLAN support: §6.3 says daily-ish reset is decay toward baseline, not a scheduled hard reset, and mood never snaps on tab-open.
   Result: grounded.

## Operational notes

- The frozen reconstruction was not modified.
- I did not read PRD files, peer slots, other waves, or non-allowlisted run artifacts.
- The audit uses only GOLD_WHYS/RUBRIC for comparison and the assigned PLAN/RECONSTRUCTION for evidence.
