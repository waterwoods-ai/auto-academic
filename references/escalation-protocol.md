# Escalation Protocol

## Tiered Review Response

| Round | Decision | Action | Review Mode for Re-review |
|-------|----------|--------|--------------------------|
| 1 | Minor | Revise all comments | `re-review` |
| 1 | Major | Revise all comments | `full` |
| 1 | Reject | Treat as Major, revise anyway | `full` |
| 2 | Minor | Targeted revision | `re-review` |
| 2 | Major | Targeted revision, switch to methodology-focus review | `methodology-focus` |
| 2 | Reject | Targeted revision, switch to methodology-focus review | `methodology-focus` |
| 3 | Minor | Final revision | `re-review` |
| 3 | Major | **ESCALATE** to inner loop | N/A (re-enters after experiments) |
| 3 | Reject | **ESCALATE** to inner loop | N/A |
| 4 | Any non-Accept | Final targeted revision | `re-review` |
| 5 | Any non-Accept | **FORCE-FINALIZE** with quality report | N/A |

## Escalation Procedure

When escalation is triggered (round 3 + Major/Reject):

### Step 1: Concern Extraction
The `escalation-analyst` agent reads:
- All reviewer reports from the current round
- The editorial decision letter
- The revision roadmap

It produces a **concern map**:

| Concern ID | Reviewer | Category | Description | Affected Hypothesis |
|-----------|----------|----------|-------------|-------------------|
| C1 | Reviewer 2 | sample_size | N=50 insufficient for effect size d=0.3 | H1 |
| C2 | Devil's Advocate | alternative_explanation | Confound: prior experience not controlled | H2 |

### Step 2: Experiment Refinement
For each concern mapped to a hypothesis:
- Load the original experiment spec
- Modify parameters to address the concern (e.g., increase sample size, add control condition)
- Create a new experiment spec version

### Step 3: Inner Loop Re-entry
Re-enter the inner loop at Phase 3 (Experiment Execution):
- Run only the modified experiments (not all experiments from scratch)
- Evaluate results with the new specs
- Update the claim-evaluation report

### Step 4: Targeted Paper Update
After inner loop re-converges:
- Re-write ONLY the sections affected by the re-run experiments (methodology, results, discussion)
- Do NOT rewrite introduction, literature review, or conclusion unless the revised hypothesis fundamentally changes the paper's framing

### Step 5: Re-enter Outer Loop
- Run integrity check on updated paper
- Re-enter peer review at round 4

## Force-Finalize

When round 5 is reached without Accept:
1. Compile the best version of the paper (from the revision with the most favorable review)
2. Generate a **quality report** documenting:
   - Unresolved reviewer concerns
   - Why they remain unresolved (experiment limitations, scope constraints)
   - Suggestions for future work
3. Append quality report to the final paper as a supplementary note
4. Complete the pipeline with a warning in the process log

## Escalation Limits

- Maximum 1 escalation per pipeline run
- If post-escalation review still returns Major at round 4, proceed to round 5 (force-finalize) — do NOT escalate again
- Rationale: infinite escalation loops waste resources. After 1 re-experiment + 5 review rounds, the paper has reached its practical ceiling.
