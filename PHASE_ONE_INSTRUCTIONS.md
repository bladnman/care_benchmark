# PHASE_ONE_INSTRUCTIONS.md - Phase 1 planning runner

You are running phase 1 of a CARE benchmark instance. Your job is to claim a new wave folder and spawn fresh planner subagents that read the PRD and write implementation plans.

This file is safe for phase-1 contexts. Do not read other repository orientation files while running phase 1.

## Defaults

- Candidate model: `claude-opus-4.7`
- Candidate effort: `very_high`
- If the user did not give a plan count, ask once for the count, then proceed.

## What the user might say

| User says | You do |
|---|---|
| `Run 5 plans` | Claim a new wave and run 5 planner subagents. |
| `phase 1, count=5` | Claim a new wave and run 5 planner subagents. |
| `Run plans on <model>` | Use the requested model and effort if specified. |

If the user asks for work beyond phase 1, complete the phase-1 portion from this file, then switch to the later-phase instructions outside any planner subagent context.

## 1. Claim the next wave

1. Find the highest existing `runs/wave_NNN/`. The next wave number is that number plus one. If `runs/` does not exist or has no waves, use `wave_001`.
2. Create `runs/wave_NNN/` with POSIX `mkdir`. This claim is atomic. If it already exists, increment `NNN` and retry.
3. Inside the claimed wave, create `plans/`.

## 2. Write wave metadata

Write `runs/wave_NNN/METADATA.json`:

```json
{
  "wave_number": <NNN>,
  "started_at": "<real ISO 8601 UTC from date -u +%Y-%m-%dT%H:%M:%SZ>",
  "completed_at": null,
  "candidate_model_default": "<candidate model>",
  "candidate_effort_default": "<candidate effort>",
  "planned_count": <N>,
  "audit_warnings": []
}
```

Use a real timestamp. Do not use placeholders.

## 3. Spawn planner subagents in parallel

For each `MMM` from `001` to formatted `N`, create `runs/wave_NNN/plans/MMM/`, then spawn one fresh-context planner subagent. Spawn all planner subagents in parallel when your harness supports it.

### Planner subagent prompt template

```text
You are the planner for run MMM in wave NNN of the CARE benchmark. Fresh context - you have no memory of any prior run.

WORKING FOLDER: {abs_path}
YOUR ASSIGNED SLOT: runs/wave_NNN/plans/MMM/

YOUR JOB:
1. Read prd/1-START_HERE.md and every PRD file it lists.
2. Produce a comprehensive original PLAN.md per the instructions in prd/1-START_HERE.md. Save it to:
       runs/wave_NNN/plans/MMM/PLAN.md
3. Write your runtime metadata to:
       runs/wave_NNN/plans/MMM/CANDIDATE_METADATA.json
   Schema (leave fields blank/null if unknown - never fabricate):
   { "model": "...", "effort_level": "...", "temperature": null, "harness": "...", "provider": "...", "parameters": {} }

READ ALLOWLIST:
- prd/
- runs/wave_NNN/plans/MMM/

WRITE ALLOWLIST:
- runs/wave_NNN/plans/MMM/PLAN.md
- runs/wave_NNN/plans/MMM/CANDIDATE_METADATA.json

Do not read or write anything outside those allowlists. Do not ask the human. Plan from the PRD. Do not implement the product.

When done, reply with a brief summary under 200 words: plan size, metadata written, operational issues, and confirmation that you stayed within the allowlists.
```

Specify the requested candidate model and effort through your harness API.

## 4. Audit phase-1 writes

After all planners return, run:

```sh
find runs/wave_NNN -newer runs/wave_NNN/METADATA.json -type f
```

Verify each planner wrote only:

- `runs/wave_NNN/plans/MMM/PLAN.md`
- `runs/wave_NNN/plans/MMM/CANDIDATE_METADATA.json`

Append any write-scope violations to `runs/wave_NNN/METADATA.json` under `audit_warnings`.

## 5. Report phase-1 completion

Tell the user:

```text
Wave NNN phase 1 complete. <K> plans in runs/wave_NNN/plans/.
Audit warnings: <list, or "none">.
```
