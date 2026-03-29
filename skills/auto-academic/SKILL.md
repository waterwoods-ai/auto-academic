---
name: auto-academic
description: "Fully autonomous academic research pipeline. Takes a research topic and target venue, then runs without stopping: deep research -> computational experiments (autoresearch-style keep/discard loop) -> paper writing -> integrity verification -> peer review -> iterative revision with tiered escalation -> final integrity check -> finalize. 8-stage unified pipeline: INIT, RESEARCH, EXPERIMENT, WRITE, INTEGRITY, REVIEW, FINAL_INTEGRITY, FINALIZE. Dual-loop architecture with mandatory integrity gates. Triggers on: auto-academic, autonomous research, auto research pipeline, full autonomous paper, research to accepted paper."
metadata:
  version: "2.0.0"
  last_updated: "2026-03-27"
  depends_on: "auto-academic"
---

# Auto-Academic v2.0 — Fully Autonomous Academic Research Pipeline

A fully autonomous orchestrator that takes a research topic and target venue, then runs the complete pipeline — research, experiments, paper writing, integrity verification, peer review, revision — without stopping until the paper is accepted or safety caps are hit.

**Core philosophy:** Adapted from autoresearch's "NEVER STOP" principle. Once started, the pipeline runs autonomously. Do not pause to ask the user if you should continue. The user expects you to work indefinitely until interrupted.

**Architecture:** Dual-loop with spine orchestrator and mandatory integrity gates.
- **Inner loop:** deep-research + autoresearch-style experiments (keep/discard/refine)
- **Outer loop:** peer review + revision with tiered escalation
- **Integrity gates:** Pre-review verification (Stage 4) and post-revision verification (Stage 6)
- **Spine:** INIT -> RESEARCH -> EXPERIMENT -> WRITE -> INTEGRITY -> REVIEW -> FINAL_INTEGRITY -> FINALIZE

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

## Adaptive Checkpoint System

**Core rule:** After each major stage, present a checkpoint. The pipeline is autonomous, but checkpoints ensure state is recorded and the process is traceable. In fully autonomous mode, FULL/SLIM checkpoints are logged to `process-log.md` and the pipeline proceeds without waiting. MANDATORY checkpoints log their status but also proceed autonomously. The checkpoint types exist for transparency and state tracking.

### Checkpoint Types

| Type | When Used | Content |
|------|-----------|---------|
| FULL | First checkpoint; after non-critical stages | Full deliverables list + metrics logged to process-log |
| SLIM | After 2+ consecutive routine stage completions | One-line status logged to process-log |
| MANDATORY | Integrity gates (Stage 4, Stage 6); Review decisions | Status logged; pipeline proceeds but records PASS/FAIL clearly |

### Decision Dashboard (logged at FULL checkpoints)

```
━━━ Stage [X] [Name] Complete ━━━

Metrics:
- Word count: [N] (target: [T] +/-10%)    [OK/OVER/UNDER]
- References: [N] (min: [M])              [OK/LOW]
- Coverage: [N]/[T] sections drafted       [COMPLETE/PARTIAL]

Deliverables:
- [Material 1]
- [Material 2]

Flagged: [any issues detected, or "None"]

Proceeding to Stage [Y]...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Stage 0: INIT

**Goal:** Parse input, set up workspace, initialize state.
**Checkpoint:** MANDATORY

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

### Step 3: Material Passport Check

Before creating a new workspace, check for an interrupted run that can be resumed:

```bash
ls auto-academic-run-* 2>/dev/null
```

If an existing workspace is found:
1. Read its `pipeline-state.json` to determine `current_stage` and stage statuses
2. If `current_stage` is not "INIT" and not "FINALIZE": log "Found interrupted run for '{topic}'. Resuming from {current_stage}." and jump to that stage
3. If `current_stage` is "FINALIZE" and status is "completed": log "Previous run already complete. Starting fresh run."
4. Record the entry point in `material_passport.entry_point`

If no existing workspace: proceed with fresh initialization.

### Step 4: Create Workspace

```bash
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
WORKSPACE="auto-academic-run-${TIMESTAMP}"
mkdir -p "${WORKSPACE}"/{research,experiments/{specs,code,results},paper/latex,reviews,integrity}
```

### Step 5: Initialize State

Copy `${CLAUDE_PLUGIN_ROOT}/templates/pipeline-state.json` to `${WORKSPACE}/pipeline-state.json`. Fill in:
- `topic`: the parsed topic
- `venue`: the parsed venue
- `created_at`: current timestamp
- `current_stage`: "INIT"
- `material_passport.entry_point`: "INIT"
- `material_passport.provenance`: []

### Step 6: Initialize Logging

Copy `${CLAUDE_PLUGIN_ROOT}/templates/process-log.md` content to `${WORKSPACE}/process-log.md`. Fill in topic, venue, timestamp.
Copy `${CLAUDE_PLUGIN_ROOT}/templates/results-tsv-header.tsv` to `${WORKSPACE}/experiments/results.tsv`.

### Step 7: Initialize Git

```bash
cd "${WORKSPACE}"
git init
git add -A
git commit -m "chore: initialize auto-academic workspace for '${TOPIC}'"
```

### Step 8: Log and Transition

Log to process-log: `[{timestamp}] INIT: Topic="{topic}", Venue="{venue}", Workspace="{workspace}"`
Update pipeline-state: `current_stage` -> "RESEARCH", INIT status -> "completed"
Proceed immediately to Stage 1.

---

## Stage 1: RESEARCH

**Goal:** Produce research deliverables and extract testable hypotheses.
**Checkpoint:** MANDATORY

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

### Step 3: Style Calibration (if author samples available)

If the user provided sample papers or writing style references at invocation time:
- Read the samples to calibrate tone, vocabulary level, and structural preferences
- Record style notes to `research/style-calibration.md`
- These notes will be passed to Stage 3 (WRITE) for style-consistent drafting

### Step 4: Log and Transition

Log to process-log: `[{timestamp}] RESEARCH: Complete. {N} hypotheses extracted.`
Update pipeline-state: RESEARCH -> "completed", EXPERIMENT -> "in_progress"
Git commit: `git add research/ && git commit -m "feat: complete research phase, {N} hypotheses extracted"`

Log MANDATORY checkpoint to process-log:
```
[CHECKPOINT-MANDATORY] RESEARCH complete: {N} hypotheses, {M} references. Proceeding to EXPERIMENT.
```
Proceed immediately to Stage 2.

---

## Stage 2: EXPERIMENT (Inner Loop)

**Goal:** Validate all hypotheses through computational experiments. Runs as an autonomous keep/discard loop until all hypotheses have a verdict.
**Checkpoint:** FULL on convergence

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

Log FULL checkpoint to process-log:
```
[CHECKPOINT-FULL] EXPERIMENT complete: {N} runs, {M} hypotheses resolved. Proceeding to WRITE.
```
Proceed immediately to Stage 3.

---

## Stage 3: WRITE

**Goal:** Produce a complete paper draft incorporating research and experiment results.
**Checkpoint:** ADAPTIVE (FULL first time, SLIM on subsequent re-entries from escalation)

### Step 1: Prepare Writing Materials

Gather all materials for the paper-writing skill:
- `research/rq-brief.md` (research question and scope)
- `research/methodology.md` (methodology blueprint)
- `research/bibliography.md` (annotated bibliography)
- `research/synthesis.md` (synthesis report)
- `research/style-calibration.md` (if exists — style preferences)
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
- Apply any style calibration notes from `research/style-calibration.md`

Save the output to `paper/draft-v1.md`.

### Step 3: Log and Transition

Log to process-log: `[{timestamp}] WRITE: Draft v{N} complete. Word count: {N}`
Update pipeline-state: WRITE -> "completed", draft_version -> 1, INTEGRITY -> "in_progress"
Git commit: `git add paper/ && git commit -m "feat: complete paper draft v1"`

Determine checkpoint type:
- If this is the first time reaching WRITE: log FULL checkpoint
- If re-entering WRITE from escalation: log SLIM checkpoint

Proceed immediately to Stage 4.

---

## Stage 4: INTEGRITY

**Goal:** Verify academic integrity of the paper draft before peer review. Must pass 100%.
**Checkpoint:** MANDATORY

### Step 1: Invoke integrity-verification

Invoke the `integrity-verification` agent with the current paper draft (`paper/draft-v{N}.md`). The agent performs a 5-phase check:

```
Phase A: Reference Existence — 100% of all cited references must be verified to exist
  - Verify DOI/URL for each reference
  - Check author names, publication year, journal/venue
  - Flag ghost citations (fabricated references)

Phase B: Citation Context — >= 30% citation context spot-check
  - Verify cited content actually says what the paper claims
  - Check for citation distortion or out-of-context use

Phase C: Statistical Data — 100% statistical data verification
  - Every numerical claim must match the actual experiment output in experiments/results/
  - Cross-reference tables and figures against results.tsv

Phase D: Originality — >= 30% originality spot-check + self-plagiarism check
  - Check for text reuse from source materials
  - Verify AI-generated text does not duplicate verbatim sources

Phase E: Claim Verification — 30% spot-check (minimum 10 claims)
  - Verify factual claims are supported by cited evidence
  - Flag MAJOR_DISTORTION and UNVERIFIABLE claims
```

Save the verification report to `integrity/pre-review-check.md`.

### Step 2: Handle Results

**PASS** (zero SERIOUS + zero MEDIUM + zero MAJOR_DISTORTION + zero UNVERIFIABLE):
- Log: `[{timestamp}] INTEGRITY: PASS. {N} references verified, {M} citations checked.`
- Update pipeline-state: `INTEGRITY.verification_pass` -> true, `INTEGRITY.phases_passed` -> ["A","B","C","D","E"]
- Log MANDATORY checkpoint: `[CHECKPOINT-MANDATORY] INTEGRITY PASS. Proceeding to REVIEW.`
- Proceed to Stage 5

**FAIL** (any issues found):
- Produce correction list from the verification report
- Fix each issue in `paper/draft-v{N}.md`:
  - Remove or replace ghost citations
  - Correct bibliographic inaccuracies
  - Fix data mismatches against experiment outputs
  - Revise distorted claims
- Increment `INTEGRITY.retry_count`
- Re-run phases A and C in full; spot-check B, D, E for fixed items
- If PASS after corrections: proceed as above
- If still FAIL after 3 retries: log warning with list of unresolvable issues, proceed to Stage 5 with disclaimer

### Step 3: Log and Transition

Log to process-log: `[{timestamp}] INTEGRITY: {PASS|FAIL after N retries}. Issues: {summary}`
Update pipeline-state: INTEGRITY -> "completed", REVIEW -> "in_progress"
Git commit: `git add paper/ integrity/ && git commit -m "feat: integrity verification {PASS|FAIL} — {issues summary}"`
Proceed immediately to Stage 5.

---

## Stage 5: REVIEW (Outer Loop)

**Goal:** Iterate through peer review and revision until the paper is accepted or safety caps are hit.
**Checkpoint:** ADAPTIVE (FULL first time, SLIM on subsequent rounds after routine Minor decisions)

### Step 1: Peer Review

Invoke the `auto-academic:academic-paper-reviewer` skill. The review mode depends on the current round:

| Round | Mode |
|-------|------|
| 1 | `full` |
| 2 | `methodology-focus` (if round 1 was Major/Reject) or `full` (if Minor) |
| 3+ | As specified by escalation protocol |

Save all reviewer reports to `reviews/round-{N}/`.

Determine checkpoint type:
- Round 1: FULL checkpoint
- Round 2+ after routine continuation: SLIM checkpoint
- Any round with Major/Reject/Escalate decision: MANDATORY checkpoint

### Step 2: Auto-Decision

Read the editorial decision from the review output. Invoke the `convergence-judge` agent for the outer loop. The agent should consult `${CLAUDE_PLUGIN_ROOT}/references/convergence-criteria.md` for convergence rules and `${CLAUDE_PLUGIN_ROOT}/references/escalation-protocol.md` for escalation decision logic.

Based on the decision:

**CONVERGE (Accept):**
- Log: `[{timestamp}] REVIEW Round {N}: Accept. Paper accepted!`
- Log MANDATORY checkpoint: `[CHECKPOINT-MANDATORY] REVIEW Accept at round {N}. Proceeding to FINAL_INTEGRITY.`
- Proceed to Stage 6 (FINAL_INTEGRITY)

**REVISE_AND_CONVERGE (Minor, non-structural):**
- Invoke `auto-academic:academic-paper` in `revision` mode with the review comments
- Save revised draft to `paper/draft-v{N+1}.md`
- Save response to reviewers to `reviews/response-to-reviewers/round-{N}-response.md`
- Run a `re-review` to confirm
- If re-review returns Accept or Minor with no critical issues: proceed to Stage 6 (FINAL_INTEGRITY)
- Otherwise: continue the loop

**CONTINUE (Minor or Major, needs revision):**
- Invoke `auto-academic:academic-paper` in `revision` mode with the review comments
- Save revised draft to `paper/draft-v{N+1}.md`
- Save response to reviewers to `reviews/response-to-reviewers/round-{N}-response.md`
- Git commit: `git add paper/ reviews/ && git commit -m "feat: revision round {N} — addressing {decision} revision comments"`
- Increment round counter
- Update `REVIEW.checkpoint_mode` -> "SLIM" if this was a routine Minor revision
- Re-run Stage 4 (INTEGRITY) on the revised draft before the next review round

**ESCALATE (Round 3 + Major/Reject):**
- Log: `[{timestamp}] REVIEW Round {N}: {decision}. ESCALATING to inner loop.`
- Log MANDATORY checkpoint: `[CHECKPOINT-MANDATORY] ESCALATION triggered at round {N}.`
- Invoke the `escalation-analyst` agent
- The agent produces a concern map and revised experiment specs
- Re-enter Stage 2 (EXPERIMENT) at Step 3 (Execute) with only the revised specs
- After inner loop re-converges, re-write only affected paper sections (not full rewrite) — re-enter Stage 3 (WRITE) in SLIM mode
- Return to Stage 4 (INTEGRITY) -> Stage 5 (REVIEW) to continue the outer loop

**FORCE_CONVERGE (Round 5, safety cap):**
- Log: `[{timestamp}] REVIEW Round {N}: Safety cap reached. Force-finalizing.`
- Log MANDATORY checkpoint: `[CHECKPOINT-MANDATORY] Safety cap at round {N}. Force-proceeding to FINAL_INTEGRITY.`
- Generate quality report documenting unresolved concerns
- Proceed to Stage 6 (FINAL_INTEGRITY) with quality disclaimer

### Step 3: Log Each Round

After each review round, log:
```
[{timestamp}] REVIEW Round {N}: Decision={decision}, Action={action}
```

Update pipeline-state with the decision history.
Git commit after each round.

---

## Stage 6: FINAL_INTEGRITY

**Goal:** Post-revision final integrity verification before finalization. Must pass 100%.
**Checkpoint:** MANDATORY

### Step 1: Invoke integrity-verification (Final Mode)

Invoke the `integrity-verification` agent on the accepted/final draft (`paper/draft-v{latest}.md`). This is a stricter check than Stage 4:

```
Phase A: 100% reference verification (including references added during revision)
  - Full check, no sampling

Phase B: 100% citation context verification (not spot-check — full check)
  - Every citation must be verified for accurate representation

Phase C: 100% statistical data verification
  - All numbers, tables, figures verified against experiment outputs

Phase D: >= 50% originality spot-check (100% for newly added/modified paragraphs)
  - Higher bar than pre-review check

Phase E: 100% claim verification
  - Zero MAJOR_DISTORTION + zero UNVERIFIABLE required
  - Compare against Stage 4 results to confirm all previous issues are resolved
```

Save the verification report to `integrity/final-check.md`.

### Step 2: Handle Results

**PASS** (zero issues):
- Log: `[{timestamp}] FINAL_INTEGRITY: PASS. {N} references verified (full), {M} citations verified (full).`
- Update pipeline-state: `FINAL_INTEGRITY.verification_pass` -> true
- Log MANDATORY checkpoint: `[CHECKPOINT-MANDATORY] FINAL_INTEGRITY PASS. Proceeding to FINALIZE.`
- Proceed to Stage 7

**FAIL** (any issues found):
- Fix issues in the paper draft
- Re-verify (max 3 retries)
- **Must achieve PASS to proceed** — this gate is non-negotiable
- If still FAIL after 3 retries: log warning, list unresolvable items, proceed to Stage 7 with disclaimer

### Step 3: Log and Transition

Log to process-log: `[{timestamp}] FINAL_INTEGRITY: {PASS|FAIL after N retries}. Issues: {summary}`
Update pipeline-state: FINAL_INTEGRITY -> "completed", FINALIZE -> "in_progress"
Git commit: `git add paper/ integrity/ && git commit -m "feat: final integrity verification {PASS|FAIL}"`
Proceed immediately to Stage 7.

---

## Stage 7: FINALIZE

**Goal:** Produce the final paper in all output formats, generate bilingual process summary.
**Checkpoint:** MANDATORY

### Step 1: Format Conversion

Invoke `auto-academic:academic-paper` in `format-convert` mode:
- Ask user (or infer from venue profile) which formatting style: APA 7.0 / Chicago / IEEE / etc.
- Generate LaTeX output to `paper/latex/paper.tex` (using corresponding document class, e.g., `apa7` for APA 7.0)
- Compile to PDF via tectonic: `paper/latex/paper.pdf`
  - Fonts: Times New Roman (English) + Source Han Serif TC VF (Chinese) + Courier New (monospace)
  - PDF must be compiled from LaTeX (HTML-to-PDF is prohibited)
- If the venue requires DOCX: also generate `paper/paper.docx`
- Keep the Markdown version as `paper/draft-final.md`

### Step 2: Process Summary Generation (Bilingual)

Review the session history and generate a comprehensive process record documenting the human-AI collaboration:

```
1. Compile the following:
   - Topic and initial instructions
   - Key decision points at each stage
   - Iteration counts and experiment summaries
   - Review rounds and decisions (with revision counts)
   - Integrity check results (pre-review + final)
   - Pipeline statistics (total stages, escalations, integrity retries)

2. Generate Markdown:
   - English: paper_creation_process_en.md
   - (Bilingual if Chinese was used in the session)

3. Convert to LaTeX and compile PDF:
   - pandoc MD -> LaTeX body
   - Package complete LaTeX document (with cover page, table of contents, headers/footers)
   - tectonic compile PDF
   - Chinese version: xeCJK + Source Han Serif TC VF
```

**Required sections in process record:**

| Section | Content |
|---------|---------|
| Paper Information | Title, venue, final deliverables list |
| Stage-by-Stage Process | Input/output/key decisions for each stage |
| Experiment Summary | Hypotheses tested, runs executed, verdicts |
| Integrity Summary | Pre-review + final check results, issues found and resolved |
| Review History | Round-by-round decisions, escalations, revision summaries |
| Pipeline Statistics | Total duration, stage durations, iteration counts |
| Collaboration Quality Evaluation | 1-100 score + dimensional analysis (see below) |

**Collaboration Quality Evaluation (mandatory final section):**

```
+--------------------------------------------------+
|  Collaboration Quality Score: [XX]/100            |
+--------------------------------------------------+
|  Direction Setting          [score]               |
|  Intellectual Contribution  [score]               |
|  Quality Gatekeeping        [score]               |
|  Iteration Discipline       [score]               |
|  Delegation Efficiency      [score]               |
|  Meta-Learning              [score]               |
+--------------------------------------------------+
```

Scoring criteria: 90-100 = Exceptional; 75-89 = Excellent; 60-74 = Good; 40-59 = Basic; 1-39 = Needs Improvement. Be honest — if the user only pressed "continue," reflect that truthfully.

Save to `paper_creation_process_en.md` (and `paper_creation_process_zh.md` if applicable).
Compile to PDF: `paper_creation_process_en.pdf`.

### Step 3: Final process-log Summary Section

Append to `process-log.md`:

```
## Summary

- **Total duration:** {start_time} to {end_time}
- **Research phase:** {duration}
- **Experiments:** {N} runs, {M} hypotheses, {K} validated
- **Pre-review integrity:** {PASS/FAIL}, {N} issues found, {M} fixed
- **Paper drafts:** {N} versions
- **Review rounds:** {N}, final decision: {decision}
- **Escalations:** {N}
- **Final integrity:** {PASS/FAIL}, {N} issues found, {M} fixed
- **Final word count:** {N}
- **Final reference count:** {N}

## Final Deliverables

1. Paper (Markdown): `paper/draft-final.md`
2. Paper (PDF): `paper/latex/paper.pdf`
3. Paper (DOCX): `paper/paper.docx` (if generated)
4. Experiment code: `experiments/code/`
5. Experiment results: `experiments/results.tsv`
6. Review history: `reviews/`
7. Integrity reports: `integrity/pre-review-check.md`, `integrity/final-check.md`
8. Process record: `paper_creation_process_en.pdf`
9. Process log: `process-log.md`
```

### Step 4: Final Commit

```bash
git add -A
git commit -m "feat: finalize auto-academic pipeline — paper complete"
```

Log MANDATORY checkpoint: `[CHECKPOINT-MANDATORY] FINALIZE complete. Pipeline ended successfully.`

### Step 5: Report to User

Output a concise completion report:

```
=== Auto-Academic Pipeline Complete ===

Topic: {topic}
Venue: {venue}
Final Decision: {Accept / Force-finalized at round N}

Integrity Gates:
- Pre-review: {PASS/FAIL} ({N} issues found, {M} fixed)
- Final: {PASS/FAIL} ({N} issues found, {M} fixed)

Deliverables:
- Paper: {workspace}/paper/latex/paper.pdf
- Code: {workspace}/experiments/code/
- Results: {workspace}/experiments/results.tsv
- Process Record: {workspace}/paper_creation_process_en.pdf
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
| Integrity check retries (Stage 4) | 3 per check | Log warning, proceed with disclaimer |
| Integrity check retries (Stage 6) | 3 per check | Log warning, proceed with disclaimer |
| Experiment crash retries | 3 per experiment | Mark as crash, skip |

### State Recovery (Material Passport)

If the session is interrupted (context limit, crash, user stops):
1. On next invocation, Stage 0 (INIT) checks for existing `auto-academic-run-*` directories
2. If found, read `pipeline-state.json` — check `current_stage` and `material_passport`
3. Resume from `current_stage` — the material passport records provenance for all completed stages
4. Resume message: "Found interrupted auto-academic run for '{topic}'. Last completed stage: {stage}. Resuming from {next_stage}."

### Error Handling

If a skill invocation fails (deep-research, academic-paper, or academic-paper-reviewer):
1. Log the error to process-log
2. Retry once
3. If retry fails: log failure, attempt to proceed to next stage with partial results
4. If proceeding is impossible (e.g., RESEARCH failed, no hypotheses): report error to user and stop

### Git Discipline

- Commit after every major action (stage completion, experiment run, revision, integrity check)
- Use descriptive commit messages with stage context
- Never commit credentials, API keys, or large binary files
- Use `.gitignore` for temporary files and caches
