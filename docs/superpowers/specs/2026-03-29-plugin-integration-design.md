# Design: Integrate academic-research-skills + superpowers into auto-academic

## Summary

Make auto-academic a self-contained Claude Code plugin by integrating all skills from academic-research-skills (4 skills, 32 agents) and superpowers (14 skills, 1 agent) into a single plugin. Consolidate agents, references, and templates at the top level. Merge auto-academic's dual-loop orchestrator with academic-pipeline's integrity verification into a unified 8-stage pipeline.

## Design Decisions

| Decision | Choice | Alternatives Considered |
|----------|--------|------------------------|
| Integration scope | Full — all skills from both repos | Minimal (only direct deps), full academic + minimal superpowers |
| Namespace | Flat (`auto-academic:*`) | Preserve origin grouping, functional grouping |
| License | Single license + NOTICE file with attribution | Per-directory licenses, defer |
| SessionStart hook | Keep, adapt for auto-academic | Drop, replace with CLAUDE.md |
| Orchestrator | Merge auto-academic + academic-pipeline | Auto-academic stays primary, keep both separate |
| File structure | Consolidate by type (agents/, references/, templates/) | Copy & rewire, monoskill |

## Unified File Layout

```
auto-academic/
├── .claude-plugin/
│   └── plugin.json                          # Updated manifest (v2.0.0)
├── commands/
│   └── auto-academic.md                     # Main entry point (existing)
├── hooks/
│   ├── hooks.json                           # SessionStart config
│   └── session-start                        # Adapted skill discovery script
├── skills/
│   ├── auto-academic/
│   │   └── SKILL.md                         # Unified 8-stage orchestrator (merged)
│   ├── deep-research/
│   │   └── SKILL.md                         # From academic-research-skills
│   ├── academic-paper/
│   │   └── SKILL.md                         # From academic-research-skills
│   ├── academic-paper-reviewer/
│   │   └── SKILL.md                         # From academic-research-skills
│   ├── academic-pipeline/
│   │   └── SKILL.md                         # Kept as standalone skill (manual workflow)
│   ├── brainstorming/
│   │   ├── SKILL.md                         # From superpowers
│   │   └── visual-companion.md              # Supporting file stays with skill
│   ├── writing-plans/
│   │   ├── SKILL.md
│   │   └── plan-document-reviewer-prompt.md
│   ├── executing-plans/
│   │   └── SKILL.md
│   ├── dispatching-parallel-agents/
│   │   └── SKILL.md
│   ├── subagent-driven-development/
│   │   ├── SKILL.md
│   │   ├── implementer-prompt.md
│   │   ├── spec-reviewer-prompt.md
│   │   └── code-quality-reviewer-prompt.md
│   ├── using-superpowers/
│   │   └── SKILL.md
│   ├── using-git-worktrees/
│   │   └── SKILL.md
│   ├── finishing-a-development-branch/
│   │   └── SKILL.md
│   ├── systematic-debugging/
│   │   └── SKILL.md
│   ├── test-driven-development/
│   │   └── SKILL.md
│   ├── verification-before-completion/
│   │   └── SKILL.md
│   ├── requesting-code-review/
│   │   └── SKILL.md
│   ├── receiving-code-review/
│   │   └── SKILL.md
│   └── writing-skills/
│       └── SKILL.md
├── agents/                                  # ALL agents consolidated (40 total)
│   ├── # auto-academic (4)
│   ├── experiment-planner.md
│   ├── experiment-evaluator.md
│   ├── convergence-judge.md
│   ├── escalation-analyst.md
│   ├── # deep-research (13)
│   ├── research-question.md
│   ├── research-architect.md
│   ├── bibliography.md
│   ├── source-verification.md
│   ├── synthesis.md
│   ├── report-compiler.md
│   ├── editor-in-chief.md
│   ├── devils-advocate.md
│   ├── ethics-review.md
│   ├── research-socratic-mentor.md          # Renamed: collision with academic-paper
│   ├── risk-of-bias.md
│   ├── meta-analysis.md
│   ├── monitoring.md
│   ├── # academic-paper (12)
│   ├── intake.md
│   ├── literature-strategist.md
│   ├── structure-architect.md
│   ├── argument-builder.md
│   ├── draft-writer.md
│   ├── citation-compliance.md
│   ├── abstract-bilingual.md
│   ├── peer-reviewer.md
│   ├── formatter.md
│   ├── paper-socratic-mentor.md             # Renamed: collision with deep-research
│   ├── visualization.md
│   ├── revision-coach.md
│   ├── # academic-paper-reviewer (7)
│   ├── field-analyst.md
│   ├── eic.md
│   ├── methodology-reviewer.md
│   ├── domain-reviewer.md
│   ├── perspective-reviewer.md
│   ├── devils-advocate-reviewer.md
│   ├── editorial-synthesizer.md
│   ├── # academic-pipeline (3)
│   ├── pipeline-orchestrator.md
│   ├── state-tracker.md
│   ├── integrity-verification.md
│   └── # superpowers (1)
│   └── code-reviewer.md
├── references/                              # ALL references consolidated (~48 files)
│   ├── # auto-academic (4)
│   ├── venue-profiles.md
│   ├── experiment-patterns.md
│   ├── convergence-criteria.md
│   ├── escalation-protocol.md
│   ├── # deep-research (14, minus duplicates)
│   ├── apa7-guide.md                        # MERGED: deep-research + academic-paper APA guides
│   ├── source-quality-hierarchy.md
│   ├── methodology-patterns.md
│   ├── logical-fallacies.md
│   ├── ethics-checklist.md
│   ├── interdisciplinary-bridges.md
│   ├── socratic-questioning-framework.md
│   ├── failure-paths.md                     # MERGED: deep-research + academic-paper
│   ├── mode-selection-guide.md              # MERGED: deep-research + academic-paper
│   ├── irb-decision-tree.md
│   ├── equator-reporting-guidelines.md
│   ├── preregistration-guide.md
│   ├── systematic-review-toolkit.md
│   ├── literature-monitoring-strategies.md
│   ├── # academic-paper (13, minus duplicates)
│   ├── abstract-writing-guide.md
│   ├── academic-writing-style.md
│   ├── apa7-chinese-citation-guide.md       # Kept separate — specialized content
│   ├── citation-format-switcher.md
│   ├── credit-authorship-guide.md
│   ├── funding-statement-guide.md
│   ├── hei-domain-glossary.md
│   ├── journal-submission-guide.md
│   ├── latex-template-reference.md
│   ├── paper-structure-patterns.md
│   ├── statistical-visualization-standards.md
│   ├── writing-quality-check.md
│   ├── # academic-paper-reviewer (5)
│   ├── editorial-decision-standards.md
│   ├── quality-rubrics.md
│   ├── review-criteria-framework.md
│   ├── statistical-reporting-standards.md
│   ├── top-journals-by-field.md
│   ├── # academic-pipeline (4)
│   ├── claim-verification-protocol.md
│   ├── mode-advisor.md
│   ├── pipeline-state-machine.md
│   ├── plagiarism-detection-protocol.md
│   └── team-collaboration-protocol.md
├── templates/                               # ALL templates consolidated (~26 files)
│   ├── # auto-academic (7 including program-spine.md)
│   ├── pipeline-state.json
│   ├── experiment-spec.md
│   ├── claim-evaluation.md
│   ├── process-log.md
│   ├── results-tsv-header.tsv
│   ├── program-spine.md                     # NEW from program.md design
│   ├── # deep-research (6)
│   ├── research-brief.md
│   ├── literature-matrix.md
│   ├── evidence-assessment.md
│   ├── preregistration.md
│   ├── prisma-protocol.md
│   ├── prisma-report.md
│   ├── # academic-paper (10)
│   ├── bilingual-abstract.md
│   ├── case-study.md
│   ├── conference-paper.md
│   ├── credit-statement.md
│   ├── funding-statement.md
│   ├── imrad.md
│   ├── literature-review.md
│   ├── policy-brief.md
│   ├── revision-tracking.md
│   ├── theoretical-paper.md
│   ├── # academic-paper-reviewer (3)
│   ├── editorial-decision.md
│   ├── peer-review-report.md
│   ├── revision-response.md
│   └── # academic-pipeline (1)
│       └── pipeline-status.md
├── examples/                                # Organized by origin
│   ├── auto-academic/
│   │   ├── education-research.md
│   │   ├── cs-ml-research.md
│   │   └── mixed-methods.md
│   ├── deep-research/
│   │   └── (7 examples)
│   ├── academic-paper/
│   │   └── (8 examples)
│   ├── academic-paper-reviewer/
│   │   └── (2 examples)
│   └── academic-pipeline/
│       └── (3 examples + showcase/)
├── shared/
│   ├── handoff-schemas.md
│   └── style-calibration-protocol.md
├── NOTICE                                   # Upstream attribution
└── LICENSE
```

## Agent Naming Convention

Normalize all agents to kebab-case without `_agent` suffix.

**Only collision:** `socratic_mentor_agent` exists in both deep-research and academic-paper.
- deep-research version → `research-socratic-mentor.md`
- academic-paper version → `paper-socratic-mentor.md`

All other 38 agents have unique names after kebab-case normalization.

## Reference Deduplication

Three files exist in both deep-research and academic-paper:

| Content | Sources | Resolution |
|---------|---------|------------|
| APA 7.0 style guide | `apa7_style_guide.md` + `apa7_extended_guide.md` | Merge → `apa7-guide.md` (extended version) |
| Failure paths | `failure_paths.md` × 2 | Merge → `failure-paths.md` (sections per skill) |
| Mode selection | `mode_selection_guide.md` × 2 | Merge → `mode-selection-guide.md` |

`apa7_chinese_citation_guide.md` is kept as a separate file — it's specialized content, not a duplicate.

After deduplication: ~48 reference files (down from ~52).

## Merged Orchestrator: 8-Stage Pipeline

Auto-academic's dual-loop pipeline merged with academic-pipeline's integrity verification and adaptive checkpoints.

```
Stage 0: INIT
  - Configuration, workspace setup, venue profile selection
  - Material passport check (from academic-pipeline: can resume mid-pipeline)
  - State tracker agent initializes pipeline-state.json

Stage 1: RESEARCH
  - Invoke deep-research skill (full mode)
  - Extract testable hypotheses → research/hypotheses.md
  - CHECKPOINT: MANDATORY (review hypotheses before experiments)

Stage 2: EXPERIMENT (inner loop — autonomous)
  - experiment-planner generates program.md per hypothesis
  - Each hypothesis runs autonomously (NEVER STOP, human interrupts)
  - After human stops: experiment-evaluator judges claim support
  - convergence-judge gates exit
  - If not converged: refine specs, re-enter loop

Stage 3: WRITE
  - Invoke academic-paper skill (full mode)
  - Style calibration if author samples available
  - CHECKPOINT: ADAPTIVE

Stage 4: INTEGRITY (from academic-pipeline)
  - Invoke integrity-verification agent
  - 5-phase check: references, citations, data, statistics, claims
  - CHECKPOINT: MANDATORY — must pass 100% before review
  - If fail: targeted fix, re-verify (max 3 retries)

Stage 5: REVIEW (outer loop)
  - Invoke academic-paper-reviewer skill (full mode)
  - Read editorial decision
  - If Accept → Stage 7
  - If Minor Revision → invoke academic-paper (revision mode), re-review
  - If Major/Reject → escalation-analyst at round 3+
    - Escalation can re-enter Stage 2 with revised experiment specs
  - Max 5 review rounds
  - CHECKPOINT: ADAPTIVE (FULL first time, SLIM on subsequent rounds)

Stage 6: FINAL INTEGRITY (from academic-pipeline)
  - Post-revision integrity re-verification
  - Must pass before finalization
  - CHECKPOINT: MANDATORY

Stage 7: FINALIZE
  - Format conversion (LaTeX/PDF/DOCX/Markdown) via academic-paper (format-convert mode)
  - Process summary generation (bilingual, from academic-pipeline)
  - CHECKPOINT: MANDATORY (final approval)
```

### What's Merged from academic-pipeline

| Feature | Source | Integrated Into |
|---------|--------|----------------|
| Integrity verification agent | academic-pipeline | Stages 4 + 6 |
| Adaptive checkpoints (FULL/SLIM/MANDATORY) | academic-pipeline | All stages |
| Process summary generation | academic-pipeline | Stage 7 |
| Material passport (mid-entry resume) | academic-pipeline | Stage 0 |
| State tracker agent | academic-pipeline | All stages |

### What's Dropped from academic-pipeline

| Feature | Reason |
|---------|--------|
| Pipeline orchestrator agent | Auto-academic SKILL.md handles orchestration directly |
| Separate RE-REVIEW / RE-REVISE stages | Folded into Stage 5 outer loop |
| 10-stage linear pipeline | Replaced by 8-stage dual-loop |
| Manual checkpoint-driven workflow | Replaced by autonomous experiment loop |

### academic-pipeline as Standalone Skill

`academic-pipeline` SKILL.md is still available as `auto-academic:academic-pipeline` for users who want the original manual checkpoint-driven workflow without autonomous experiments. It operates independently of the merged orchestrator.

## SessionStart Hook

Adapted from superpowers. Same mechanism, updated paths.

**hooks/hooks.json:**
```json
{
  "hooks": {
    "SessionStart": [
      {
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/hooks/session-start"
      }
    ]
  }
}
```

The `session-start` script reads `skills/using-superpowers/SKILL.md` and injects skill discovery instructions. Updated to use `${CLAUDE_PLUGIN_ROOT}` pointing to auto-academic's root.

## plugin.json

```json
{
  "name": "auto-academic",
  "description": "Autonomous academic research pipeline: deep research, computational experiments, paper writing, peer review, and iterative revision",
  "version": "2.0.0",
  "author": {
    "name": "Tom"
  }
}
```

## NOTICE

```
This plugin integrates code from the following projects:

academic-research-skills
  Author: Cheng-I Wu
  License: CC BY-NC 4.0
  Source: https://github.com/Cheng-I-Wu/academic-research-skills
  Skills: deep-research, academic-paper, academic-paper-reviewer, academic-pipeline

superpowers
  Author: Jesse Vincent
  License: See original repository
  Source: https://github.com/obra/superpowers
  Skills: brainstorming, writing-plans, executing-plans, dispatching-parallel-agents,
          and 10 additional development workflow skills
```

## Path Reference Updates

All files that reference other plugin files must be updated:

1. **SKILL.md files**: Update skill invocations from `deep-research` → `auto-academic:deep-research`, `superpowers:writing-plans` → `auto-academic:writing-plans`, etc.
2. **Agent files**: Update `${CLAUDE_PLUGIN_ROOT}` references from upstream paths to consolidated paths (e.g., `skills/auto-academic/references/` → `references/`)
3. **Cross-references between agents**: Update from `_agent` suffix names to kebab-case names

## Integration with program.md Design

The previously approved program.md design (2026-03-29-program-md-design.md) integrates into this structure as:

- `templates/program-spine.md` — the fixed spine template
- `references/experiment-patterns.md` — updated with program.md examples per pattern
- `agents/experiment-planner.md` — updated to generate program.md + initial code
- Runtime output at `experiments/programs/{hypothesis_id}/`

No conflicts between the two designs. The program.md design slots into Stage 2 of the merged pipeline.

## autoresearch/ Directory

The `autoresearch/` directory (containing `program.md` and `train.py` sample code) remains in the plugin as a reference implementation. It is not part of the plugin's auto-discovered structure — it serves as documentation for the autonomous experiment loop mechanism that `program-spine.md` generalizes.
