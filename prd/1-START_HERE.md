# 1-START_HERE.md — Planning instructions (phase 1)

You are running phase 1 of the CARE benchmark: produce a comprehensive plan from this product spec (PRD). The orchestrator that spawned you has assigned you a write slot under `runs/wave_NNN/plans/MMM/` and given you read/write allowlists. This file is your guide for **what to plan** — the substantive content of the `PLAN.md` you produce.

Read this file fully, then read every PRD file listed below.

## Files to read (every one of them)

- `product_brief.md` — headline concept, design philosophy, voice and tone, what the product is and isn't
- `concepts.md` — domain vocabulary (read this second; the other files lean on its definitions)
- `bird_engine.md` — the bird-evolution mechanic: personality, mood, calls, drift
- `interactions.md` — what a session looks like; greeting, listen-in, offer, settle, the field notebook, presence
- `aviary_layout.md` — the visual scene, layout rules, day/night, weather, top-bar chrome
- `accounts_sync.md` — auth, server-side simulation tick, multi-device sync, conflict handling, privacy
- `social_optional.md` — visit invitations and what they deliberately are not
- `accessibility_perf.md` — accessibility surfaces and performance budgets
- `non_goals.md` — explicit out-of-scope statements

## What to produce — `PLAN.md`

A comprehensive implementation plan, detailed enough that a separate engineering team could execute it without further clarification. Cover, at minimum:

- **Scope** — what is and isn't in v1, with the non-goals respected
- **Architecture** — service shape, client/server split, render pipeline boundary
- **Data model** — birds, personality vectors, mood, presence events, notebook entries, accounts
- **API surface** — how clients pull state, how clients submit interaction events, how the visit-invitation flow works
- **Simulation engine design** — the server-side tick, drift function, mood transitions, call-grammar runtime
- **Sync model** — how a single canonical aviary state propagates to multiple devices, how conflicts are prevented
- **Frontend rendering pipeline** — scene composition, idle micro-motion, transitions, reduced-motion mode
- **Audio pipeline** — procedural call synthesis, chorus mixing, listen-in mix decay, the WebAudio fallback
- **Accessibility surfaces** — screen-reader narration, captions, focus and keyboard navigation, contrast
- **Performance budgets and observability** — bundle size, time-to-first-bird, runtime budgets, what we measure and what we deliberately don't
- **Rollout** — how we ship v1, how we ramp birds-per-aviary, what we instrument from day one
- **Risks** — what could go wrong, especially around drift calibration, sync correctness, audio uncanniness, and accessibility regressions

The plan is for a frontier engineering team. Be specific. Don't restate the spec — interpret it into an executable plan. **Do NOT implement the product.** The deliverable is the plan only.

## Also produce — `CANDIDATE_METADATA.json`

A small JSON file with your runtime metadata. Your orchestrator's prompt told you the path. Schema (leave fields blank/null if unknown — never fabricate):

```json
{
  "model": "<your model identifier, e.g. 'claude-opus-4.7'>",
  "effort_level": "<low | medium | high | very_high | max>",
  "temperature": null,
  "harness": "<harness name, e.g. 'claude-code', 'cursor-cli'>",
  "provider": "<provider, e.g. 'anthropic'>",
  "parameters": {}
}
```

## Constraints

- **No questions back to the human.** Plan from what the PRD provides. If something is ambiguous, make a defensible call and note it in your plan.
- **Stay inside the read and write allowlists your orchestrator gave you.** Do not read peer slots, other waves, root orientation files, archive files, or any file outside the allowlist.
- **Write exactly two files** in your assigned slot: `PLAN.md` and `CANDIDATE_METADATA.json`. Nothing else.

## When done

Your orchestrator will receive your summary. Reply per its instructions — typically under 200 words covering plan size, runtime metadata you wrote, any operational issues, and confirmation you stayed within the allowlists.
