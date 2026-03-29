# Experiment: {{HYPOTHESIS_ID}} — {{HYPOTHESIS_TITLE}}

## Setup

Create git branch `experiment/{{HYPOTHESIS_ID}}` from main.
Initialize `results.tsv` with header: `commit\tstatus\ttier\tdescription\tevaluation_summary`
Read all in-scope files listed below for context.

## Scope

{{FREEFORM: Planner generates this section}}

### What you CAN modify
- `main.py` — {{description of experiment code}}

### Tier 1 Tunable Parameters
- {{param_1}}: {{description, current value, range}}
- {{param_2}}: {{description, current value, range}}
- ...

### Tier 2 Code Change Boundaries
- {{what structural changes are allowed}}
- {{what algorithms/methods can be swapped}}

### What you CANNOT modify
- {{read-only files: data, instruments, etc.}}

### Dependencies
- {{language}}: {{library_1}}, {{library_2}}, ...

## Evaluation

{{FREEFORM: Planner generates this section}}

### A run is BETTER if:
- {{criterion_1}}
- {{criterion_2}}
- ...

### A run is WORSE if:
- {{criterion_1}}
- {{criterion_2}}
- ...

### Hypothesis Drift Check
- The experiment must still test: "{{original hypothesis statement}}"
- If a modification changes WHAT is being tested (not just HOW), it is drift.
- Drift = automatic discard regardless of metric improvement.

### Ambiguity Guidance
- {{edge cases specific to this experiment type}}

## Output Format

The experiment script MUST print results in this exact format:

```
---
{{metric_1}}:     {{example_value}}
{{metric_2}}:     {{example_value}}
runtime_sec:      {{example_value}}
status:           success
```

## Logging

Results are logged to `results.tsv` (tab-separated, NOT committed to git).

| Column | Description |
|--------|------------|
| commit | Short git hash (7 chars) of the experiment commit |
| status | `keep`, `discard`, or `crash` |
| tier | `param` (Tier 1) or `code` (Tier 2) |
| description | What this iteration tried (one line) |
| evaluation_summary | Brief LLM judgment rationale (one line) |

## The Experiment Loop

LOOP FOREVER:

1. Look at git state: current branch, current commit.
2. Decide what to try next. Respect the current tier constraints.
3. Modify code accordingly.
4. `git commit -am "experiment: {{brief description}}"`
5. Run: `python main.py > run.log 2>&1`
6. Read results from run.log. Parse the output format above.
7. Evaluate using the criteria in the Evaluation section.
   - Also run the Hypothesis Drift Check.
8. If BETTER and no drift: KEEP the commit. Branch advances.
9. If WORSE or DRIFT: `git reset --hard HEAD~1`. Branch reverts.
10. Append one row to `results.tsv`. Do NOT commit results.tsv.
11. Tier escalation check:
    - Count the last 5 consecutive results.
    - If all 5 are `discard` AND current tier is `param`:
      escalate to `code` tier. Log: "Escalating to Tier 2 (code changes)."
    - The planner may set this threshold lower (minimum 3) in the Scope section.

### Crash Handling

If the run crashes (non-zero exit code):
1. Read `tail -n 50 run.log` for diagnosis.
2. If fixable (typo, missing import, wrong path): fix and retry. Max 3 retries per crash.
3. If fundamental (OOM, incompatible library, missing data): log as `crash` in results.tsv, revert, move on to next idea.

### Timeout

If a run exceeds 10 minutes, kill the process and treat as a crash.

### NEVER STOP

Do NOT pause to ask the human if you should continue.
Do NOT stop after N iterations.
The loop runs until the human interrupts you, period.
