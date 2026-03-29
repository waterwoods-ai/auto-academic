---
name: auto-academic
description: "Fully autonomous academic research pipeline. Takes a research topic and target venue, then runs without stopping: deep research -> computational experiments (autoresearch-style keep/discard loop) -> paper writing -> peer review -> iterative revision with tiered escalation -> loops until the paper is accepted or safety caps are hit. Dual-loop architecture: inner loop (research + experiments) and outer loop (review + revise). Triggers on: auto-academic, autonomous research, auto research pipeline, full autonomous paper, research to accepted paper."
metadata:
  version: "1.0.0"
  last_updated: "2026-03-26"
  depends_on: "auto-academic"
---

# Auto-Academic v1.0 — Fully Autonomous Academic Research Pipeline

A fully autonomous orchestrator that takes a research topic and target venue, then runs the complete pipeline — research, experiments, paper writing, peer review, revision — without stopping until the paper is accepted or safety caps are hit.

**Core philosophy:** Adapted from autoresearch's "NEVER STOP" principle. Once started, the pipeline runs autonomously. Do not pause to ask the user if you should continue. The user expects you to work indefinitely until interrupted.

**Architecture:** Dual-loop with spine orchestrator.
- **Inner loop:** deep-research + autoresearch-style experiments (keep/discard/refine)
- **Outer loop:** peer review + revision with tiered escalation
- **Spine:** INIT -> RESEARCH -> EXPERIMENT -> WRITE -> REVIEW -> FINALIZE

## Trigger Conditions

**English:** auto-academic, autonomous research, auto research pipeline, full autonomous paper, research to accepted paper, run auto-academic

## Dependencies

All required skills are bundled within this plugin. No external plugin dependencies.

## Invocation

The user provides:
1. **Topic** (required): the research subject
2. **Venue** (required): target publication venue

Examples:
```
/auto-academic "Impact of LLMs on undergraduate writing skills" --venue "Computers & Education"
Run auto-academic on the effects of AI tutoring in STEM education, targeting the Internet and Higher Education
```

---

## Stage 0: INIT

**Goal:** Parse input, set up workspace, initialize state.

### Step 1: Parse Input

Extract topic and venue from the user's message. If either is missing, infer from context or use defaults:
- Missing venue: default to APA 7.0 format, Tier 3 rigor (SSCI standard)
- Missing topic: this is an error — the pipeline cannot start without a topic

### Step 2: Venue Profile Lookup

Read `${CLAUDE_PLUGIN_ROOT}/references/venue-profiles.md`. Match the user's venue to a profile category. If no match, use WebSearch to research the venue, then assign the closest category.

Record the venue profile settings:
- Citation style
- Paper format
- Page limit
- Review rigor tier
- Experiment expectations

### Step 3: Create Workspace

```bash
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
WORKSPACE="auto-academic-run-${TIMESTAMP}"
mkdir -p "${WORKSPACE}"/{research,experiments/{specs,code,results},paper/latex,reviews,integrity}
```

### Step 4: Initialize State

Copy `${CLAUDE_PLUGIN_ROOT}/templates/pipeline-state.json` to `${WORKSPACE}/pipeline-state.json`. Fill in:
- `topic`: the parsed topic
- `venue`: the parsed venue
- `created_at`: current timestamp
- `current_stage`: "INIT"

### Step 5: Initialize Logging

Copy `${CLAUDE_PLUGIN_ROOT}/templates/process-log.md` content to `${WORKSPACE}/process-log.md`. Fill in topic, venue, timestamp.
Copy `${CLAUDE_PLUGIN_ROOT}/templates/results-tsv-header.tsv` to `${WORKSPACE}/experiments/results.tsv`.

### Step 6: Initialize Git

```bash
cd "${WORKSPACE}"
git init
git add -A
git commit -m "chore: initialize auto-academic workspace for '${TOPIC}'"
```

### Step 7: Log and Transition

Log to process-log: `[{timestamp}] INIT: Topic="{topic}", Venue="{venue}", Workspace="{workspace}"`
Update pipeline-state: `current_stage` -> "RESEARCH", INIT status -> "completed"
Proceed immediately to Stage 1.

---

## Stage 1: RESEARCH

**Goal:** Produce research deliverables and extract testable hypotheses.

### Step 1: Invoke deep-research

Trigger the `auto-academic:deep-research` skill in `full` mode with the topic. Provide:
- The parsed topic as the research question seed
- The venue profile for calibrating research depth

Wait for deep-research to complete. It produces:
- RQ Brief -> save to `research/rq-brief.md`
- Methodology Blueprint -> save to `research/methodology.md`
- Annotated Bibliography -> save to `research/bibliography.md`
- Synthesis Report -> save to `research/synthesis.md`

### Step 2: Extract Hypotheses

Read the synthesis report. Identify all claims that:
1. Are testable through computational experiments
2. Have measurable outcomes
3. Are central to the paper's argument

For each claim, formulate a hypothesis:
- State it in testable form (e.g., "Students using AI tutoring will show higher test scores than control group")
- Identify the independent and dependent variables
- Specify the expected direction of effect

Save to `research/hypotheses.md` using this format:

```
## Hypotheses

### H1: {statement}
- **Variables:** IV = {X}, DV = {Y}
- **Expected direction:** {positive / negative / nonlinear}
- **Derived from:** Synthesis Report, Section {N}
- **Evidence base:** {key citations supporting this hypothesis}

### H2: {statement}
...
```

### Step 3: Log and Transition

Log to process-log: `[{timestamp}] RESEARCH: Complete. {N} hypotheses extracted.`
Update pipeline-state: RESEARCH -> "completed", EXPERIMENT -> "in_progress"
Git commit: `git add research/ && git commit -m "feat: complete research phase, {N} hypotheses extracted"`
Proceed immediately to Stage 2.

---

## Stage 2: EXPERIMENT (Inner Loop)

**Goal:** Validate all hypotheses through computational experiments. Runs as an autonomous keep/discard loop until all hypotheses have a verdict.

### Step 1: Plan Experiments

Invoke the `experiment-planner` agent with:
- `research/hypotheses.md`
- `research/methodology.md`
- Venue profile experiment expectations
- `${CLAUDE_PLUGIN_ROOT}/references/experiment-patterns.md`

The agent produces one experiment spec per hypothesis, saved to `experiments/specs/`.

### Step 2: Implement Experiments

For each experiment spec, invoke the `auto-academic:writing-plans` skill to create an implementation plan, then invoke `auto-academic:executing-plans` to implement the code.

If multiple experiments are independent (no shared data dependencies), use `auto-academic:dispatching-parallel-agents` to implement them concurrently.

Each experiment's code is saved to `experiments/code/{hypothesis_id}/`.

### Step 3: Execute Experiments (The Loop)

Run each experiment. For each run:

1. Execute the experiment code, redirecting all output:
   ```bash
   cd experiments/code/{hypothesis_id}
   python main.py > ../../results/run-{NNN}/output.log 2>&1
   echo $? > ../../results/run-{NNN}/exit_code.txt
   ```

2. Check the exit code:
   - Exit 0: proceed to evaluation
   - Non-zero: mark as CRASH

3. Read the output and parse metrics.

4. Log to `experiments/results.tsv`:
   ```
   {run_number}\t{hypothesis_id}\t{status}\t{metrics}\t{description}
   ```

5. Git commit:
   ```bash
   git add experiments/
   git commit -m "experiment: run {NNN} for {hypothesis_id} — {status}"
   ```

### Step 4: Handle Crashes

If an experiment crashes:
1. Read the last 50 lines of the log to diagnose the error
2. If it's a simple fix (typo, import error, wrong path): fix and retry
3. If it's a fundamental issue (OOM, incompatible library, impossible computation): skip, log "crash" status
4. Maximum 3 retry attempts per experiment

### Step 5: Evaluate Results

After each successful run, invoke the `experiment-evaluator` agent with:
- The experiment spec
- The experiment output
- The run log

The agent updates `experiments/claim-evaluation.md`.

### Step 6: Check Convergence

After evaluating results, invoke the `convergence-judge` agent for the inner loop. The agent should consult `${CLAUDE_PLUGIN_ROOT}/references/convergence-criteria.md` for detailed convergence rules.
- If `CONVERGE`: exit the inner loop, proceed to WRITE
- If `CONTINUE`: identify which hypothesis needs re-running, refine the experiment spec, go back to Step 3
- If `FORCE_CONVERGE`: log warning, proceed to WRITE with best results

### Step 7: Log and Transition

Log to process-log: `[{timestamp}] EXPERIMENT: Inner loop converged after {N} runs. Verdicts: {summary}`
Update pipeline-state: EXPERIMENT -> "completed", WRITE -> "in_progress"
Git commit: `git add experiments/ && git commit -m "feat: complete experiment phase, all hypotheses resolved"`
Proceed immediately to Stage 3.

---

## Stage 3: WRITE

**Goal:** Produce a complete paper draft incorporating research and experiment results.

### Step 1: Prepare Writing Materials

Gather all materials for the paper-writing skill:
- `research/rq-brief.md` (research question and scope)
- `research/methodology.md` (methodology blueprint)
- `research/bibliography.md` (annotated bibliography)
- `research/synthesis.md` (synthesis report)
- `experiments/claim-evaluation.md` (hypothesis-evidence mapping)
- `experiments/results/` (raw experiment outputs for tables and figures)
- Venue profile (citation style, format, page limit)

### Step 2: Invoke academic-paper

Trigger the `auto-academic:academic-paper` skill in `full` mode. Provide all gathered materials and instruct it to:
- Use the venue's citation style and format
- Include experiment methodology in the Methods section
- Include experiment results with tables/figures in the Results section
- Ground the Discussion in the actual experiment outcomes (not hypothetical)
- Reference the claim-evaluation for each argument in the Discussion

Save the output to `paper/draft-v1.md`.

### Step 3: Log and Transition

Log to process-log: `[{timestamp}] WRITE: Draft v1 complete. Word count: {N}`
Update pipeline-state: WRITE -> "completed", draft_version -> 1, REVIEW -> "in_progress"
Git commit: `git add paper/ && git commit -m "feat: complete paper draft v1"`
Proceed immediately to Stage 4.

---

## Stage 4: REVIEW (Outer Loop)

**Goal:** Iterate through peer review and revision until the paper is accepted or safety caps are hit.

### Step 1: Integrity Check

Before submitting for review, run the `integrity_verification_agent` (from academic-pipeline):
- Verify 100% of citations are real and correctly formatted
- Verify experiment data in the paper matches actual outputs in `experiments/results/`
- Verify all claims in the paper are supported by the claim-evaluation report

If integrity check fails:
- Auto-fix citations and data references
- Re-verify (max 3 rounds)
- Save report to `integrity/pre-review-check.md`

If integrity check passes after 3 rounds but still has issues: log warning, proceed anyway.

### Step 2: Peer Review

Invoke the `auto-academic:academic-paper-reviewer` skill. The review mode depends on the current round:

| Round | Mode |
|-------|------|
| 1 | `full` |
| 2 | `methodology-focus` (if round 1 was Major/Reject) or `full` (if Minor) |
| 3+ | As specified by escalation protocol |

Save all reviewer reports to `reviews/round-{N}/`.

### Step 3: Auto-Decision

Read the editorial decision from the review output. Invoke the `convergence-judge` agent for the outer loop. The agent should consult `${CLAUDE_PLUGIN_ROOT}/references/convergence-criteria.md` for convergence rules and `${CLAUDE_PLUGIN_ROOT}/references/escalation-protocol.md` for escalation decision logic.

Based on the decision:

**CONVERGE (Accept):**
- Log: `[{timestamp}] REVIEW Round {N}: Accept. Paper accepted!`
- Proceed to Stage 5 (FINALIZE)

**REVISE_AND_CONVERGE (Minor, non-structural):**
- Invoke `auto-academic:academic-paper` in `revision` mode with the review comments
- Save revised draft to `paper/draft-v{N+1}.md`
- Save response to reviewers to `reviews/response-to-reviewers/round-{N}-response.md`
- Run a `re-review` to confirm
- If re-review returns Accept or Minor with no critical issues: proceed to FINALIZE
- Otherwise: continue the loop

**CONTINUE (Minor or Major, needs revision):**
- Invoke `auto-academic:academic-paper` in `revision` mode with the review comments
- Save revised draft to `paper/draft-v{N+1}.md`
- Save response to reviewers to `reviews/response-to-reviewers/round-{N}-response.md`
- Git commit: `git add paper/ reviews/ && git commit -m "feat: revision round {N} — addressing {decision} revision comments"`
- Increment round counter
- Go back to Step 1 (integrity check on revised draft)

**ESCALATE (Round 3 + Major/Reject):**
- Log: `[{timestamp}] REVIEW Round {N}: {decision}. ESCALATING to inner loop.`
- Invoke the `escalation-analyst` agent
- The agent produces a concern map and revised experiment specs
- Re-enter Stage 2 (EXPERIMENT) at Step 3 (Execute) with only the revised specs
- After inner loop re-converges, re-write only affected paper sections (not full rewrite)
- Return to Stage 4 Step 1 (integrity check) to continue the outer loop

**FORCE_CONVERGE (Round 5, safety cap):**
- Log: `[{timestamp}] REVIEW Round {N}: Safety cap reached. Force-finalizing.`
- Generate quality report documenting unresolved concerns
- Proceed to Stage 5 (FINALIZE) with quality disclaimer

### Step 4: Log Each Round

After each review round, log:
```
[{timestamp}] REVIEW Round {N}: Decision={decision}, Action={action}
```

Update pipeline-state with the decision history.
Git commit after each round.

---

## Stage 5: FINALIZE

**Goal:** Produce the final paper in all output formats, run final integrity check, generate process summary.

### Step 1: Final Integrity Check

Run `integrity_verification_agent` one final time on the accepted/force-finalized draft:
- Must achieve 100% pass for citations and data accuracy
- If any issues: auto-fix and re-verify (max 3 rounds)
- Save report to `integrity/final-check.md`

### Step 2: Format Conversion

Invoke `auto-academic:academic-paper` in `format-convert` mode:
- Generate LaTeX output to `paper/latex/paper.tex`
- Compile to PDF: `paper/latex/paper.pdf`
- If the venue requires DOCX: also generate `paper/paper.docx`
- Keep the Markdown version as `paper/draft-final.md`

### Step 3: Process Summary

Generate `process-log.md` final section with:

```
## Summary

- **Total duration:** {start_time} to {end_time}
- **Research phase:** {duration}
- **Experiments:** {N} runs, {M} hypotheses, {K} validated
- **Paper drafts:** {N} versions
- **Review rounds:** {N}, final decision: {decision}
- **Escalations:** {N}
- **Final word count:** {N}
- **Final reference count:** {N}

## Final Deliverables

1. Paper: `paper/draft-final.md` + `paper/latex/paper.pdf`
2. Experiment code: `experiments/code/`
3. Experiment results: `experiments/results.tsv`
4. Review history: `reviews/`
5. Process log: `process-log.md`
```

### Step 4: Final Commit

```bash
git add -A
git commit -m "feat: finalize auto-academic pipeline — paper complete"
```

### Step 5: Report to User

Output a concise completion report:

```
=== Auto-Academic Pipeline Complete ===

Topic: {topic}
Venue: {venue}
Final Decision: {Accept / Force-finalized at round N}

Deliverables:
- Paper: {workspace}/paper/latex/paper.pdf
- Code: {workspace}/experiments/code/
- Results: {workspace}/experiments/results.tsv
- Full log: {workspace}/process-log.md

{If force-finalized: "Note: Quality report attached — see integrity/final-check.md for unresolved concerns."}
```

---

## Safety Rules

### NEVER STOP

Once the pipeline starts (after INIT), do NOT:
- Ask the user if you should continue
- Pause to check if the user is still watching
- Say "should I proceed to the next stage?"
- Wait for confirmation between stages

The pipeline runs autonomously until completion or user interruption. The user may be away from the computer.

### Safety Caps (Non-Negotiable)

| Cap | Value | Action on Hit |
|-----|-------|--------------|
| Inner loop experiments | 10 runs | Force-converge, proceed with best results |
| Outer loop review rounds | 5 rounds | Force-finalize with quality report |
| Escalation re-entries | 1 | No second escalation, proceed to final rounds |
| Integrity check retries | 3 per check | Log warning, proceed |
| Experiment crash retries | 3 per experiment | Mark as crash, skip |

### State Recovery

If the session is interrupted (context limit, crash, user stops):
1. On next invocation, check for existing `auto-academic-run-*` directories
2. If found, read `pipeline-state.json` to determine where the pipeline stopped
3. Offer to resume from the last completed stage
4. Resume message: "Found interrupted auto-academic run for '{topic}'. Last completed stage: {stage}. Resuming from {next_stage}."

### Error Handling

If a skill invocation fails (deep-research, academic-paper, or academic-paper-reviewer):
1. Log the error to process-log
2. Retry once
3. If retry fails: log failure, attempt to proceed to next stage with partial results
4. If proceeding is impossible (e.g., RESEARCH failed, no hypotheses): report error to user and stop

### Git Discipline

- Commit after every major action (stage completion, experiment run, revision)
- Use descriptive commit messages with stage context
- Never commit credentials, API keys, or large binary files
- Use `.gitignore` for temporary files and caches
