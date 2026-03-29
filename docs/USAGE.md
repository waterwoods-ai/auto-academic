# Auto-Academic Usage Guide

A complete guide to using the auto-academic plugin — from installation to running your first autonomous research pipeline.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [How the Pipeline Works](#how-the-pipeline-works)
- [Invocation](#invocation)
- [Venue Tiers](#venue-tiers)
- [The 8-Stage Pipeline](#the-8-stage-pipeline)
- [The Experiment Loop](#the-experiment-loop)
- [Bundled Skills](#bundled-skills)
- [Workspace Output](#workspace-output)
- [Resuming Interrupted Runs](#resuming-interrupted-runs)
- [Safety Caps](#safety-caps)
- [Monitoring Progress](#monitoring-progress)
- [Using Individual Skills](#using-individual-skills)
- [Troubleshooting](#troubleshooting)

---

## Installation

### From Marketplace (recommended)

```bash
/plugin marketplace add waterwoods-ai/auto-academic
/plugin install auto-academic@waterwoods-ai
/reload-plugins
```

### From Source

```bash
git clone https://github.com/waterwoods-ai/auto-academic.git
cp -r auto-academic/ ~/.claude/plugins/auto-academic/
```

No external plugin dependencies. Everything is bundled.

---

## Quick Start

```
/auto-academic "Impact of LLMs on undergraduate writing skills" --venue "Computers & Education"
```

That's it. The pipeline runs autonomously from research to finalized paper. You can walk away — it will not stop to ask you questions.

---

## How the Pipeline Works

Auto-academic runs a fully autonomous 8-stage pipeline with two loops:

```
INIT → RESEARCH → EXPERIMENT → WRITE → INTEGRITY → REVIEW → FINAL_INTEGRITY → FINALIZE
         ↑                                           |
         └── inner loop (autonomous) ────────────────┘
                                                     |
                                              REVIEW ←→ REVISE (outer loop)
```

**Inner loop** (Stages 1-2): Deep research produces hypotheses. Computational experiments validate them using an autoresearch-style keep/discard mechanism. A convergence judge exits the loop when all hypotheses have evidence.

**Outer loop** (Stage 5): Simulated peer review produces revision requests. The paper is revised and re-reviewed. If a Major/Reject decision persists at round 3, an escalation analyst sends specific concerns back to the inner loop for targeted re-experimentation.

**Integrity gates** (Stages 4 and 6): Before and after review, a 5-phase verification checks every reference, citation, statistic, and claim in the paper.

---

## Invocation

### Via slash command

```
/auto-academic "your research topic" --venue "Target Journal"
```

### Via natural language

```
Run auto-academic on the effects of AI tutoring in STEM education, targeting Internet and Higher Education
```

### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| topic | Yes | — | The research question or area |
| venue | No | APA 7.0, Tier 3 (SSCI) | Target publication venue |

### Examples

```
/auto-academic "Does retrieval practice improve long-term retention in online courses?" --venue "Computers & Education"

/auto-academic "Transformer architectures for low-resource NLP" --venue "ACL"

/auto-academic "Agent-based modeling of misinformation spread" --venue "Nature Human Behaviour"

/auto-academic "Effectiveness of gamification in K-12 math education"
```

The last example uses the default venue (Tier 3 SSCI journal with APA 7.0 formatting).

---

## Venue Tiers

The pipeline adapts rigor, format, and experiment expectations to your target venue:

| Tier | Examples | Citation | Experiment Expectations |
|------|----------|----------|----------------------|
| 1 | NeurIPS, ICML, ACL, CVPR | Conference style | Ablations, multiple seeds, SOTA comparison, reproducible code |
| 2 | Nature, Science | Numbered | Pre-registration, effect sizes + CI, replication checks |
| 3 | Computers & Education, Internet & Higher Education | APA 7.0 | Survey validation, effect sizes, IRB mention, mixed methods OK |
| 4 | Regional journals (JRES, Taiwan TESOL) | APA 7.0 | Appropriate methodology, standard rigor |
| 5 | Workshop/conference proceedings | Varies | Preliminary results acceptable |

If your venue isn't in the database, the pipeline researches it via web search and assigns the closest tier.

---

## The 8-Stage Pipeline

### Stage 0: INIT

Parses your topic and venue, looks up the venue profile, creates a timestamped workspace directory, initializes git tracking, and sets up pipeline state.

If an interrupted run for the same topic is detected, offers to resume from where it stopped.

### Stage 1: RESEARCH

Invokes the `deep-research` skill (13-agent team) in full mode. Produces:
- Research question brief
- Methodology blueprint
- Annotated bibliography
- Synthesis report
- Testable hypotheses (H1, H2, ...)

### Stage 2: EXPERIMENT (Inner Loop)

For each hypothesis:
1. The **experiment-planner** designs an experiment spec and generates a `program.md` (autonomous loop instructions)
2. Code is implemented and executed
3. The **experiment-evaluator** judges whether results support the hypothesis
4. The **convergence-judge** decides: continue experimenting or move on

The experiment loop uses an autoresearch-style mechanism: modify code, run, evaluate (LLM-judged keep/discard), and iterate. Parameters are tuned first; if that plateaus, the agent escalates to code-level changes.

### Stage 3: WRITE

Invokes the `academic-paper` skill (12-agent team) to produce a complete draft. Uses all research materials, experiment results, and venue-specific formatting. Optionally applies style calibration if you provided sample papers.

### Stage 4: INTEGRITY (Pre-Review)

The **integrity-verification** agent runs 5 checks on the draft:

| Phase | Check | Coverage |
|-------|-------|----------|
| A | Reference existence (DOI, authors, year) | 100% |
| B | Citation context accuracy | 30% spot-check |
| C | Statistical data vs. experiment output | 100% |
| D | Originality / self-plagiarism | 30% spot-check |
| E | Claim-evidence alignment | 30% spot-check |

Must pass to proceed. Up to 3 retry rounds with targeted fixes.

### Stage 5: REVIEW (Outer Loop)

Invokes the `academic-paper-reviewer` skill (7-agent team with 5 independent reviewers). Handles the editorial decision:

| Decision | Action |
|----------|--------|
| Accept | Proceed to final integrity check |
| Minor Revision | Revise, re-verify integrity, re-review |
| Major Revision | Revise, re-verify, re-review. At round 3: escalate |
| Reject | At round 3: escalate to re-experimentation |

**Escalation**: The escalation-analyst extracts methodology concerns from reviewer reports, maps them to specific hypotheses, and re-enters the experiment loop with revised specs. Only the affected paper sections are rewritten.

### Stage 6: FINAL INTEGRITY (Post-Revision)

Stricter than Stage 4 — 100% coverage on all phases (no spot-checking). The paper must have zero issues to proceed.

### Stage 7: FINALIZE

- Converts the paper to LaTeX and compiles to PDF
- Generates DOCX if the venue requires it
- Produces a bilingual process summary documenting the entire pipeline run
- Records collaboration quality evaluation (6 dimensions, 1-100 score)

---

## The Experiment Loop

The experiment loop adapts autoresearch's mechanism for general academic research. Instead of always training ML models, it works across 7 experiment patterns:

1. **Survey/questionnaire analysis** — statistical tests on survey data
2. **Text/content analysis** — NLP, coding, thematic analysis
3. **ML classification/regression** — predictive modeling
4. **Simulation/agent-based modeling** — system dynamics
5. **Statistical comparison/meta-analysis** — effect size synthesis
6. **Network/graph analysis** — relational structures
7. **Natural experiment/quasi-experimental** — causal inference

### How it works

For each hypothesis, the experiment-planner generates a `program.md` file containing:

- **Scope**: What the agent can modify (parameters vs. code)
- **Evaluation criteria**: LLM-judged — what makes a run "better" or "worse"
- **Hypothesis drift check**: Ensures modifications still test the original hypothesis

The loop runs autonomously:
1. Modify experiment code
2. Git commit
3. Run the experiment
4. Evaluate results (LLM-judged keep/discard)
5. If better: keep the commit. If worse: revert.
6. Repeat.

### Tiered iteration

- **Tier 1 (default)**: Only tune parameters (thresholds, sample sizes, model hyperparameters)
- **Tier 2 (escalation)**: Modify code structure (swap algorithms, change preprocessing)

Escalation happens after 5 consecutive discards at Tier 1.

---

## Bundled Skills

Auto-academic bundles 19 skills. You can use any of them independently:

### Academic Skills

| Skill | Command | Description |
|-------|---------|-------------|
| auto-academic | `/auto-academic` | Full autonomous pipeline |
| deep-research | `auto-academic:deep-research` | 13-agent research team, 7 modes |
| academic-paper | `auto-academic:academic-paper` | 12-agent paper writing, 8 modes |
| academic-paper-reviewer | `auto-academic:academic-paper-reviewer` | 5-reviewer peer review simulation |
| academic-pipeline | `auto-academic:academic-pipeline` | Manual checkpoint-driven pipeline (non-autonomous) |

### Development Skills (from superpowers)

| Skill | Description |
|-------|-------------|
| brainstorming | Design exploration before implementation |
| writing-plans | Convert specs to bite-sized implementation plans |
| executing-plans | Execute plans with checkpoints |
| dispatching-parallel-agents | Run independent tasks concurrently |
| subagent-driven-development | Fresh subagent per task + two-stage review |
| using-git-worktrees | Isolated branch management |
| finishing-a-development-branch | Merge/PR/cleanup decisions |
| systematic-debugging | Root cause investigation |
| test-driven-development | RED-GREEN-REFACTOR enforcement |
| verification-before-completion | Verify before claiming done |
| requesting-code-review | Pre-merge review |
| receiving-code-review | Handle review feedback |
| writing-skills | Create new skills |
| using-superpowers | Skill discovery (injected at session start) |

---

## Workspace Output

Each pipeline run creates a timestamped workspace:

```
auto-academic-run-20260329-143000/
├── pipeline-state.json          # Stage tracking, iteration counts
├── process-log.md               # Timestamped log of all actions
├── research/
│   ├── rq-brief.md              # Research question brief
│   ├── methodology.md           # Methodology blueprint
│   ├── bibliography.md          # Annotated bibliography
│   ├── synthesis.md             # Synthesis report
│   ├── hypotheses.md            # Testable hypotheses (H1, H2, ...)
│   └── style-calibration.md     # (if author samples provided)
├── experiments/
│   ├── specs/                   # One spec per hypothesis
│   ├── programs/                # program.md + code per hypothesis
│   │   └── h1/
│   │       ├── program.md       # Autonomous loop instructions
│   │       ├── main.py          # Experiment code
│   │       ├── results.tsv      # Keep/discard log (untracked)
│   │       └── run.log          # Latest run output
│   ├── code/                    # Implemented experiment code
│   ├── results/                 # Run outputs (run-001/, run-002/, ...)
│   ├── results.tsv              # Summary log
│   └── claim-evaluation.md      # Hypothesis-evidence mapping
├── paper/
│   ├── draft-v1.md              # First draft
│   ├── draft-v2.md              # After revision (if any)
│   ├── draft-final.md           # Accepted version
│   └── latex/
│       ├── paper.tex            # LaTeX source
│       └── paper.pdf            # Compiled PDF
├── reviews/
│   ├── round-1/                 # Reviewer reports
│   ├── round-2/                 # (if revision needed)
│   └── response-to-reviewers/   # Author responses
├── integrity/
│   ├── pre-review-check.md      # Stage 4 report
│   └── final-check.md           # Stage 6 report
└── paper_creation_process_en.md # Process summary
```

Everything is git-tracked within the workspace for full provenance.

---

## Resuming Interrupted Runs

If the pipeline is interrupted (context limit, crash, manual stop), it can resume:

1. Navigate to the directory containing the workspace
2. Run the same command:
   ```
   /auto-academic "same topic" --venue "same venue"
   ```
3. The pipeline detects the existing `auto-academic-run-*` directory, reads `pipeline-state.json`, and resumes from the last completed stage

This is called the **material passport** — it tracks which stages completed and what materials were produced.

---

## Safety Caps

The pipeline has hard limits to prevent infinite loops:

| Cap | Limit | What Happens |
|-----|-------|-------------|
| Experiment runs (inner loop) | Convergence-judge decides | Force-converge with best results |
| Review rounds (outer loop) | 5 rounds | Force-finalize with quality disclaimer |
| Escalations | 1 per pipeline run | No further re-experimentation after first escalation |
| Integrity retries | 3 per gate | Proceed with disclaimer listing unresolved issues |
| Crash retries (per experiment) | 3 | Skip experiment, log as crash |

---

## Monitoring Progress

While the pipeline runs, you can monitor:

### Process log
The pipeline writes to `process-log.md` in the workspace. Each stage transition, checkpoint, and decision is timestamped.

### Pipeline state
Read `pipeline-state.json` for structured state: current stage, iteration counts, hypothesis verdicts, review decisions.

### Experiment results
Check `experiments/results.tsv` for the keep/discard history of each experiment run.

### Git log
Every significant step is committed. Run `git log --oneline` inside the workspace to see the full history.

---

## Using Individual Skills

You don't have to run the full pipeline. Each bundled skill works standalone:

### Deep research only

```
Use auto-academic:deep-research to investigate "the impact of spaced repetition on vocabulary acquisition"
```

Modes: `full`, `quick`, `review`, `lit-review`, `fact-check`, `socratic`, `systematic-review`

### Write a paper from existing research

```
Use auto-academic:academic-paper in full mode to write a paper about [topic] using the research materials in ./research/
```

Modes: `full`, `plan`, `outline-only`, `revision`, `abstract-only`, `lit-review`, `format-convert`, `citation-check`

### Peer review a paper

```
Use auto-academic:academic-paper-reviewer to review the paper at ./paper/draft.md
```

Modes: `full`, `re-review`, `quick`, `methodology-focus`, `guided`

### Manual pipeline (with checkpoints)

```
Use auto-academic:academic-pipeline to run the full pipeline with human-in-the-loop checkpoints
```

This is the non-autonomous version — it pauses at each stage for your approval.

---

## Troubleshooting

### Pipeline stops unexpectedly

The pipeline follows "NEVER STOP" — it should not ask you questions once started. If it stops:
1. Check if you hit a context window limit (the pipeline is token-intensive)
2. Resume with the same command — the material passport will pick up where it left off

### Experiments keep crashing

Check `experiments/programs/{hypothesis_id}/run.log` for error details. Common causes:
- Missing Python dependencies (the experiment code uses standard libraries only)
- Data files not generated (check experiment spec for data requirements)
- OOM on large simulations

### Integrity check keeps failing

Read `integrity/pre-review-check.md` for the specific issues. Common causes:
- Ghost citations (fabricated references) — the deep-research phase sometimes generates plausible but non-existent references
- Statistical mismatches — paper claims don't match experiment output files
- Citation distortion — paper misrepresents what a source says

The pipeline auto-fixes these, but after 3 retries it proceeds with a disclaimer.

### Review loop seems stuck

Check `pipeline-state.json` — look at `REVIEW.outer_loop_round` and `REVIEW.decisions`. The pipeline will force-finalize at round 5. If you're at round 3 with Major/Reject, the escalation analyst should trigger automatic re-experimentation.

---

## Attribution

This plugin integrates code from:
- **academic-research-skills** by Cheng-I Wu (CC BY-NC 4.0)
- **superpowers** by Jesse Vincent (MIT)
- **autoresearch** mechanism by Andrej Karpathy

See the `NOTICE` file for full attribution.
