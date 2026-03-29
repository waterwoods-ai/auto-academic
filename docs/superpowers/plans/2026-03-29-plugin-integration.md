# Plugin Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Integrate academic-research-skills and superpowers into auto-academic as a self-contained Claude Code plugin with a unified 8-stage pipeline and autonomous experiment loop.

**Architecture:** Consolidate-by-type structure — all agents in `agents/`, all references in `references/`, all templates in `templates/`. Skills keep their own directories under `skills/`. Merge auto-academic and academic-pipeline orchestrators. Adapt superpowers SessionStart hook.

**Tech Stack:** Claude Code plugin system (markdown-based skills, agents, hooks). Bash for hook scripts. JSON for plugin manifest.

**Specs:**
- `docs/superpowers/specs/2026-03-29-plugin-integration-design.md`
- `docs/superpowers/specs/2026-03-29-program-md-design.md`

---

## Phase 1: Scaffold and Infrastructure

### Task 1: Update plugin.json and create NOTICE

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Create: `NOTICE`

- [ ] **Step 1: Update plugin.json**

```json
{
  "name": "auto-academic",
  "description": "Autonomous academic research pipeline: deep research, computational experiments, paper writing, peer review, and iterative revision",
  "version": "2.0.0",
  "author": {
    "name": "Tom"
  },
  "license": "MIT",
  "keywords": [
    "academic",
    "research",
    "paper",
    "autonomous",
    "experiments",
    "peer-review",
    "deep-research",
    "tdd",
    "debugging"
  ]
}
```

- [ ] **Step 2: Create NOTICE file**

```
This plugin integrates code from the following projects:

academic-research-skills
  Author: Cheng-I Wu
  License: CC BY-NC 4.0
  Source: https://github.com/Cheng-I-Wu/academic-research-skills
  Skills: deep-research, academic-paper, academic-paper-reviewer, academic-pipeline

superpowers
  Author: Jesse Vincent
  License: MIT
  Source: https://github.com/obra/superpowers
  Skills: brainstorming, writing-plans, executing-plans, dispatching-parallel-agents,
          subagent-driven-development, using-git-worktrees, finishing-a-development-branch,
          systematic-debugging, test-driven-development, verification-before-completion,
          requesting-code-review, receiving-code-review, using-superpowers, writing-skills
```

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/plugin.json NOTICE
git commit -m "chore: bump to v2.0.0, add NOTICE for upstream attribution"
```

---

### Task 2: Set up hooks infrastructure

**Files:**
- Create: `hooks/hooks.json`
- Create: `hooks/run-hook.cmd`
- Create: `hooks/session-start`

- [ ] **Step 1: Create hooks/hooks.json**

Copy from superpowers — identical structure:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 2: Create hooks/run-hook.cmd**

Copy verbatim from `/Users/tom/Project/auto-academic/superpowers/hooks/run-hook.cmd` — this is the cross-platform polyglot wrapper.

```bash
cp /Users/tom/Project/auto-academic/superpowers/hooks/run-hook.cmd hooks/run-hook.cmd
chmod +x hooks/run-hook.cmd
```

- [ ] **Step 3: Create hooks/session-start**

Copy from `/Users/tom/Project/auto-academic/superpowers/hooks/session-start` — the script reads `using-superpowers/SKILL.md` relative to plugin root, which will work once that skill is copied in Task 5.

```bash
cp /Users/tom/Project/auto-academic/superpowers/hooks/session-start hooks/session-start
chmod +x hooks/session-start
```

- [ ] **Step 4: Commit**

```bash
git add hooks/
git commit -m "feat: add SessionStart hook for skill discovery injection"
```

---

## Phase 2: Copy Skills

### Task 3: Copy academic-research-skills SKILL.md files

**Files:**
- Create: `skills/deep-research/SKILL.md`
- Create: `skills/academic-paper/SKILL.md`
- Create: `skills/academic-paper-reviewer/SKILL.md`
- Create: `skills/academic-pipeline/SKILL.md`

- [ ] **Step 1: Create skill directories and copy SKILL.md files**

```bash
mkdir -p skills/deep-research skills/academic-paper skills/academic-paper-reviewer skills/academic-pipeline

cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/SKILL.md skills/deep-research/SKILL.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/SKILL.md skills/academic-paper/SKILL.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/SKILL.md skills/academic-paper-reviewer/SKILL.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/SKILL.md skills/academic-pipeline/SKILL.md
```

- [ ] **Step 2: Commit**

```bash
git add skills/deep-research/ skills/academic-paper/ skills/academic-paper-reviewer/ skills/academic-pipeline/
git commit -m "feat: integrate academic-research-skills SKILL.md files"
```

---

### Task 4: Copy superpowers core skills (3 direct dependencies)

**Files:**
- Create: `skills/writing-plans/SKILL.md`
- Create: `skills/writing-plans/plan-document-reviewer-prompt.md`
- Create: `skills/executing-plans/SKILL.md`
- Create: `skills/dispatching-parallel-agents/SKILL.md`

- [ ] **Step 1: Copy writing-plans with supporting file**

```bash
mkdir -p skills/writing-plans
cp /Users/tom/Project/auto-academic/superpowers/skills/writing-plans/SKILL.md skills/writing-plans/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/writing-plans/plan-document-reviewer-prompt.md skills/writing-plans/plan-document-reviewer-prompt.md
```

- [ ] **Step 2: Copy executing-plans and dispatching-parallel-agents**

```bash
mkdir -p skills/executing-plans skills/dispatching-parallel-agents
cp /Users/tom/Project/auto-academic/superpowers/skills/executing-plans/SKILL.md skills/executing-plans/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/dispatching-parallel-agents/SKILL.md skills/dispatching-parallel-agents/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add skills/writing-plans/ skills/executing-plans/ skills/dispatching-parallel-agents/
git commit -m "feat: integrate superpowers core skills (writing-plans, executing-plans, dispatching-parallel-agents)"
```

---

### Task 5: Copy remaining superpowers skills

**Files:**
- Create: `skills/brainstorming/SKILL.md` + supporting files
- Create: `skills/subagent-driven-development/SKILL.md` + 3 prompt files
- Create: `skills/using-superpowers/SKILL.md` + references/
- Create: `skills/using-git-worktrees/SKILL.md`
- Create: `skills/finishing-a-development-branch/SKILL.md`
- Create: `skills/systematic-debugging/SKILL.md` + supporting files
- Create: `skills/test-driven-development/SKILL.md` + supporting file
- Create: `skills/verification-before-completion/SKILL.md`
- Create: `skills/requesting-code-review/SKILL.md` + supporting file
- Create: `skills/receiving-code-review/SKILL.md`
- Create: `skills/writing-skills/SKILL.md` + supporting files

- [ ] **Step 1: Copy brainstorming with supporting files**

```bash
mkdir -p skills/brainstorming
cp /Users/tom/Project/auto-academic/superpowers/skills/brainstorming/SKILL.md skills/brainstorming/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/brainstorming/visual-companion.md skills/brainstorming/visual-companion.md
cp /Users/tom/Project/auto-academic/superpowers/skills/brainstorming/spec-document-reviewer-prompt.md skills/brainstorming/spec-document-reviewer-prompt.md
cp -r /Users/tom/Project/auto-academic/superpowers/skills/brainstorming/scripts skills/brainstorming/scripts
```

- [ ] **Step 2: Copy subagent-driven-development with prompt files**

```bash
mkdir -p skills/subagent-driven-development
cp /Users/tom/Project/auto-academic/superpowers/skills/subagent-driven-development/SKILL.md skills/subagent-driven-development/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/subagent-driven-development/implementer-prompt.md skills/subagent-driven-development/implementer-prompt.md
cp /Users/tom/Project/auto-academic/superpowers/skills/subagent-driven-development/spec-reviewer-prompt.md skills/subagent-driven-development/spec-reviewer-prompt.md
cp /Users/tom/Project/auto-academic/superpowers/skills/subagent-driven-development/code-quality-reviewer-prompt.md skills/subagent-driven-development/code-quality-reviewer-prompt.md
```

- [ ] **Step 3: Copy using-superpowers with references**

```bash
mkdir -p skills/using-superpowers/references
cp /Users/tom/Project/auto-academic/superpowers/skills/using-superpowers/SKILL.md skills/using-superpowers/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/using-superpowers/references/codex-tools.md skills/using-superpowers/references/codex-tools.md
cp /Users/tom/Project/auto-academic/superpowers/skills/using-superpowers/references/gemini-tools.md skills/using-superpowers/references/gemini-tools.md
```

- [ ] **Step 4: Copy remaining simple skills**

```bash
mkdir -p skills/using-git-worktrees skills/finishing-a-development-branch skills/verification-before-completion skills/receiving-code-review

cp /Users/tom/Project/auto-academic/superpowers/skills/using-git-worktrees/SKILL.md skills/using-git-worktrees/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/finishing-a-development-branch/SKILL.md skills/finishing-a-development-branch/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/verification-before-completion/SKILL.md skills/verification-before-completion/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/receiving-code-review/SKILL.md skills/receiving-code-review/SKILL.md
```

- [ ] **Step 5: Copy skills with supporting files**

```bash
mkdir -p skills/systematic-debugging skills/test-driven-development skills/requesting-code-review skills/writing-skills/examples

cp /Users/tom/Project/auto-academic/superpowers/skills/systematic-debugging/SKILL.md skills/systematic-debugging/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/systematic-debugging/condition-based-waiting.md skills/systematic-debugging/condition-based-waiting.md
cp /Users/tom/Project/auto-academic/superpowers/skills/systematic-debugging/defense-in-depth.md skills/systematic-debugging/defense-in-depth.md
cp /Users/tom/Project/auto-academic/superpowers/skills/systematic-debugging/root-cause-tracing.md skills/systematic-debugging/root-cause-tracing.md
cp /Users/tom/Project/auto-academic/superpowers/skills/systematic-debugging/find-polluter.sh skills/systematic-debugging/find-polluter.sh

cp /Users/tom/Project/auto-academic/superpowers/skills/test-driven-development/SKILL.md skills/test-driven-development/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/test-driven-development/testing-anti-patterns.md skills/test-driven-development/testing-anti-patterns.md

cp /Users/tom/Project/auto-academic/superpowers/skills/requesting-code-review/SKILL.md skills/requesting-code-review/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/requesting-code-review/code-reviewer.md skills/requesting-code-review/code-reviewer.md

cp /Users/tom/Project/auto-academic/superpowers/skills/writing-skills/SKILL.md skills/writing-skills/SKILL.md
cp /Users/tom/Project/auto-academic/superpowers/skills/writing-skills/anthropic-best-practices.md skills/writing-skills/anthropic-best-practices.md
cp /Users/tom/Project/auto-academic/superpowers/skills/writing-skills/persuasion-principles.md skills/writing-skills/persuasion-principles.md
cp /Users/tom/Project/auto-academic/superpowers/skills/writing-skills/testing-skills-with-subagents.md skills/writing-skills/testing-skills-with-subagents.md
cp /Users/tom/Project/auto-academic/superpowers/skills/writing-skills/examples/CLAUDE_MD_TESTING.md skills/writing-skills/examples/CLAUDE_MD_TESTING.md
```

- [ ] **Step 6: Commit**

```bash
git add skills/brainstorming/ skills/subagent-driven-development/ skills/using-superpowers/ skills/using-git-worktrees/ skills/finishing-a-development-branch/ skills/systematic-debugging/ skills/test-driven-development/ skills/verification-before-completion/ skills/requesting-code-review/ skills/receiving-code-review/ skills/writing-skills/
git commit -m "feat: integrate remaining superpowers skills (11 skills)"
```

---

## Phase 3: Consolidate Agents

### Task 6: Copy and rename deep-research agents (13 agents)

**Files:**
- Create: `agents/research-question.md`
- Create: `agents/research-architect.md`
- Create: `agents/bibliography.md`
- Create: `agents/source-verification.md`
- Create: `agents/synthesis.md`
- Create: `agents/report-compiler.md`
- Create: `agents/editor-in-chief.md`
- Create: `agents/devils-advocate.md`
- Create: `agents/ethics-review.md`
- Create: `agents/research-socratic-mentor.md`
- Create: `agents/risk-of-bias.md`
- Create: `agents/meta-analysis.md`
- Create: `agents/monitoring.md`

- [ ] **Step 1: Copy agents with renamed filenames**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/research_question_agent.md agents/research-question.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/research_architect_agent.md agents/research-architect.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/bibliography_agent.md agents/bibliography.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/source_verification_agent.md agents/source-verification.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/synthesis_agent.md agents/synthesis.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/report_compiler_agent.md agents/report-compiler.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/editor_in_chief_agent.md agents/editor-in-chief.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/devils_advocate_agent.md agents/devils-advocate.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/ethics_review_agent.md agents/ethics-review.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/socratic_mentor_agent.md agents/research-socratic-mentor.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/risk_of_bias_agent.md agents/risk-of-bias.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/meta_analysis_agent.md agents/meta-analysis.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/agents/monitoring_agent.md agents/monitoring.md
```

- [ ] **Step 2: Commit**

```bash
git add agents/research-question.md agents/research-architect.md agents/bibliography.md agents/source-verification.md agents/synthesis.md agents/report-compiler.md agents/editor-in-chief.md agents/devils-advocate.md agents/ethics-review.md agents/research-socratic-mentor.md agents/risk-of-bias.md agents/meta-analysis.md agents/monitoring.md
git commit -m "feat: integrate deep-research agents (13 agents, kebab-case renamed)"
```

---

### Task 7: Copy and rename academic-paper agents (12 agents)

**Files:**
- Create: `agents/intake.md`
- Create: `agents/literature-strategist.md`
- Create: `agents/structure-architect.md`
- Create: `agents/argument-builder.md`
- Create: `agents/draft-writer.md`
- Create: `agents/citation-compliance.md`
- Create: `agents/abstract-bilingual.md`
- Create: `agents/peer-reviewer.md`
- Create: `agents/formatter.md`
- Create: `agents/paper-socratic-mentor.md`
- Create: `agents/visualization.md`
- Create: `agents/revision-coach.md`

- [ ] **Step 1: Copy agents with renamed filenames**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/intake_agent.md agents/intake.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/literature_strategist_agent.md agents/literature-strategist.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/structure_architect_agent.md agents/structure-architect.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/argument_builder_agent.md agents/argument-builder.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/draft_writer_agent.md agents/draft-writer.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/citation_compliance_agent.md agents/citation-compliance.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/abstract_bilingual_agent.md agents/abstract-bilingual.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/peer_reviewer_agent.md agents/peer-reviewer.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/formatter_agent.md agents/formatter.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/socratic_mentor_agent.md agents/paper-socratic-mentor.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/visualization_agent.md agents/visualization.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/agents/revision_coach_agent.md agents/revision-coach.md
```

- [ ] **Step 2: Commit**

```bash
git add agents/intake.md agents/literature-strategist.md agents/structure-architect.md agents/argument-builder.md agents/draft-writer.md agents/citation-compliance.md agents/abstract-bilingual.md agents/peer-reviewer.md agents/formatter.md agents/paper-socratic-mentor.md agents/visualization.md agents/revision-coach.md
git commit -m "feat: integrate academic-paper agents (12 agents, kebab-case renamed)"
```

---

### Task 8: Copy and rename reviewer + pipeline + superpowers agents (11 agents)

**Files:**
- Create: `agents/field-analyst.md`
- Create: `agents/eic.md`
- Create: `agents/methodology-reviewer.md`
- Create: `agents/domain-reviewer.md`
- Create: `agents/perspective-reviewer.md`
- Create: `agents/devils-advocate-reviewer.md`
- Create: `agents/editorial-synthesizer.md`
- Create: `agents/pipeline-orchestrator.md`
- Create: `agents/state-tracker.md`
- Create: `agents/integrity-verification.md`
- Create: `agents/code-reviewer.md`

- [ ] **Step 1: Copy academic-paper-reviewer agents (7)**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/agents/field_analyst_agent.md agents/field-analyst.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/agents/eic_agent.md agents/eic.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/agents/methodology_reviewer_agent.md agents/methodology-reviewer.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/agents/domain_reviewer_agent.md agents/domain-reviewer.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/agents/perspective_reviewer_agent.md agents/perspective-reviewer.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/agents/devils_advocate_reviewer_agent.md agents/devils-advocate-reviewer.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/agents/editorial_synthesizer_agent.md agents/editorial-synthesizer.md
```

- [ ] **Step 2: Copy academic-pipeline agents (3)**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/agents/pipeline_orchestrator_agent.md agents/pipeline-orchestrator.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/agents/state_tracker_agent.md agents/state-tracker.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/agents/integrity_verification_agent.md agents/integrity-verification.md
```

- [ ] **Step 3: Copy superpowers agent (1)**

```bash
cp /Users/tom/Project/auto-academic/superpowers/agents/code-reviewer.md agents/code-reviewer.md
```

- [ ] **Step 4: Commit**

```bash
git add agents/field-analyst.md agents/eic.md agents/methodology-reviewer.md agents/domain-reviewer.md agents/perspective-reviewer.md agents/devils-advocate-reviewer.md agents/editorial-synthesizer.md agents/pipeline-orchestrator.md agents/state-tracker.md agents/integrity-verification.md agents/code-reviewer.md
git commit -m "feat: integrate reviewer, pipeline, and superpowers agents (11 agents)"
```

---

## Phase 4: Consolidate References

### Task 9: Copy deep-research references (14 files, 3 will be merged later)

**Files:**
- Create: `references/apa7-style-guide-deep.md` (temp name, will merge in Task 12)
- Create: `references/source-quality-hierarchy.md`
- Create: `references/methodology-patterns.md`
- Create: `references/logical-fallacies.md`
- Create: `references/ethics-checklist.md`
- Create: `references/interdisciplinary-bridges.md`
- Create: `references/socratic-questioning-framework.md`
- Create: `references/failure-paths-deep.md` (temp name, will merge in Task 12)
- Create: `references/mode-selection-guide-deep.md` (temp name, will merge in Task 12)
- Create: `references/irb-decision-tree.md`
- Create: `references/equator-reporting-guidelines.md`
- Create: `references/preregistration-guide.md`
- Create: `references/systematic-review-toolkit.md`
- Create: `references/literature-monitoring-strategies.md`

- [ ] **Step 1: Copy non-duplicate references**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/source_quality_hierarchy.md references/source-quality-hierarchy.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/methodology_patterns.md references/methodology-patterns.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/logical_fallacies.md references/logical-fallacies.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/ethics_checklist.md references/ethics-checklist.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/interdisciplinary_bridges.md references/interdisciplinary-bridges.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/socratic_questioning_framework.md references/socratic-questioning-framework.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/irb_decision_tree.md references/irb-decision-tree.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/equator_reporting_guidelines.md references/equator-reporting-guidelines.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/preregistration_guide.md references/preregistration-guide.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/systematic_review_toolkit.md references/systematic-review-toolkit.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/literature_monitoring_strategies.md references/literature-monitoring-strategies.md
```

- [ ] **Step 2: Copy files that will be merged (temp names)**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/apa7_style_guide.md references/apa7-style-guide-deep.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/failure_paths.md references/failure-paths-deep.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/references/mode_selection_guide.md references/mode-selection-guide-deep.md
```

- [ ] **Step 3: Commit**

```bash
git add references/source-quality-hierarchy.md references/methodology-patterns.md references/logical-fallacies.md references/ethics-checklist.md references/interdisciplinary-bridges.md references/socratic-questioning-framework.md references/irb-decision-tree.md references/equator-reporting-guidelines.md references/preregistration-guide.md references/systematic-review-toolkit.md references/literature-monitoring-strategies.md references/apa7-style-guide-deep.md references/failure-paths-deep.md references/mode-selection-guide-deep.md
git commit -m "feat: integrate deep-research references (14 files)"
```

---

### Task 10: Copy academic-paper references (13 files, 3 will be merged)

**Files:**
- Create: `references/abstract-writing-guide.md`
- Create: `references/academic-writing-style.md`
- Create: `references/apa7-extended-guide-paper.md` (temp name, merge target)
- Create: `references/apa7-chinese-citation-guide.md`
- Create: `references/citation-format-switcher.md`
- Create: `references/credit-authorship-guide.md`
- Create: `references/failure-paths-paper.md` (temp name, merge target)
- Create: `references/funding-statement-guide.md`
- Create: `references/hei-domain-glossary.md`
- Create: `references/journal-submission-guide.md`
- Create: `references/latex-template-reference.md`
- Create: `references/mode-selection-guide-paper.md` (temp name, merge target)
- Create: `references/paper-structure-patterns.md`
- Create: `references/statistical-visualization-standards.md`
- Create: `references/writing-quality-check.md`

- [ ] **Step 1: Copy non-duplicate references**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/abstract_writing_guide.md references/abstract-writing-guide.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/academic_writing_style.md references/academic-writing-style.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/apa7_chinese_citation_guide.md references/apa7-chinese-citation-guide.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/citation_format_switcher.md references/citation-format-switcher.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/credit_authorship_guide.md references/credit-authorship-guide.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/funding_statement_guide.md references/funding-statement-guide.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/hei_domain_glossary.md references/hei-domain-glossary.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/journal_submission_guide.md references/journal-submission-guide.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/latex_template_reference.md references/latex-template-reference.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/paper_structure_patterns.md references/paper-structure-patterns.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/statistical_visualization_standards.md references/statistical-visualization-standards.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/writing_quality_check.md references/writing-quality-check.md
```

- [ ] **Step 2: Copy files that will be merged (temp names)**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/apa7_extended_guide.md references/apa7-extended-guide-paper.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/failure_paths.md references/failure-paths-paper.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/references/mode_selection_guide.md references/mode-selection-guide-paper.md
```

- [ ] **Step 3: Commit**

```bash
git add references/abstract-writing-guide.md references/academic-writing-style.md references/apa7-chinese-citation-guide.md references/citation-format-switcher.md references/credit-authorship-guide.md references/funding-statement-guide.md references/hei-domain-glossary.md references/journal-submission-guide.md references/latex-template-reference.md references/paper-structure-patterns.md references/statistical-visualization-standards.md references/writing-quality-check.md references/apa7-extended-guide-paper.md references/failure-paths-paper.md references/mode-selection-guide-paper.md
git commit -m "feat: integrate academic-paper references (13 files)"
```

---

### Task 11: Copy reviewer + pipeline references (10 files)

**Files:**
- Create: `references/editorial-decision-standards.md`
- Create: `references/quality-rubrics.md`
- Create: `references/review-criteria-framework.md`
- Create: `references/statistical-reporting-standards.md`
- Create: `references/top-journals-by-field.md`
- Create: `references/claim-verification-protocol.md`
- Create: `references/mode-advisor.md`
- Create: `references/pipeline-state-machine.md`
- Create: `references/plagiarism-detection-protocol.md`
- Create: `references/team-collaboration-protocol.md`

- [ ] **Step 1: Copy academic-paper-reviewer references (5)**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/references/editorial_decision_standards.md references/editorial-decision-standards.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/references/quality_rubrics.md references/quality-rubrics.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/references/review_criteria_framework.md references/review-criteria-framework.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/references/statistical_reporting_standards.md references/statistical-reporting-standards.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/references/top_journals_by_field.md references/top-journals-by-field.md
```

- [ ] **Step 2: Copy academic-pipeline references (5)**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/references/claim_verification_protocol.md references/claim-verification-protocol.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/references/mode_advisor.md references/mode-advisor.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/references/pipeline_state_machine.md references/pipeline-state-machine.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/references/plagiarism_detection_protocol.md references/plagiarism-detection-protocol.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/references/team_collaboration_protocol.md references/team-collaboration-protocol.md
```

- [ ] **Step 3: Commit**

```bash
git add references/editorial-decision-standards.md references/quality-rubrics.md references/review-criteria-framework.md references/statistical-reporting-standards.md references/top-journals-by-field.md references/claim-verification-protocol.md references/mode-advisor.md references/pipeline-state-machine.md references/plagiarism-detection-protocol.md references/team-collaboration-protocol.md
git commit -m "feat: integrate reviewer and pipeline references (10 files)"
```

---

### Task 12: Merge duplicate references

**Files:**
- Create: `references/apa7-guide.md` (merged from deep + paper APA guides)
- Create: `references/failure-paths.md` (merged from deep + paper failure paths)
- Create: `references/mode-selection-guide.md` (merged from deep + paper mode guides)
- Delete: 6 temp files

- [ ] **Step 1: Merge APA 7.0 guides**

Read both `references/apa7-style-guide-deep.md` and `references/apa7-extended-guide-paper.md`. Create `references/apa7-guide.md` with the extended (paper) version as the base, incorporating any unique content from the deep-research version. The paper version is more comprehensive — use it as the primary, append any deep-research-specific sections not already covered.

- [ ] **Step 2: Merge failure paths**

Read both `references/failure-paths-deep.md` and `references/failure-paths-paper.md`. Create `references/failure-paths.md` with clear section headers:
- `## Deep Research Failure Paths` (content from deep-research)
- `## Academic Paper Failure Paths` (content from academic-paper)

- [ ] **Step 3: Merge mode selection guides**

Read both `references/mode-selection-guide-deep.md` and `references/mode-selection-guide-paper.md`. Create `references/mode-selection-guide.md` with clear section headers:
- `## Deep Research Modes` (content from deep-research)
- `## Academic Paper Modes` (content from academic-paper)

- [ ] **Step 4: Remove temp files**

```bash
rm references/apa7-style-guide-deep.md references/apa7-extended-guide-paper.md
rm references/failure-paths-deep.md references/failure-paths-paper.md
rm references/mode-selection-guide-deep.md references/mode-selection-guide-paper.md
```

- [ ] **Step 5: Commit**

```bash
git add references/apa7-guide.md references/failure-paths.md references/mode-selection-guide.md
git add -u references/  # stages deletions
git commit -m "refactor: merge duplicate references (APA guide, failure paths, mode selection)"
```

---

### Task 13: Migrate existing auto-academic references

Move references from `skills/auto-academic/references/` to top-level `references/`.

**Files:**
- Move: `skills/auto-academic/references/venue-profiles.md` → `references/venue-profiles.md`
- Move: `skills/auto-academic/references/experiment-patterns.md` → `references/experiment-patterns.md`
- Move: `skills/auto-academic/references/convergence-criteria.md` → `references/convergence-criteria.md`
- Move: `skills/auto-academic/references/escalation-protocol.md` → `references/escalation-protocol.md`

- [ ] **Step 1: Move references**

```bash
mv skills/auto-academic/references/venue-profiles.md references/venue-profiles.md
mv skills/auto-academic/references/experiment-patterns.md references/experiment-patterns.md
mv skills/auto-academic/references/convergence-criteria.md references/convergence-criteria.md
mv skills/auto-academic/references/escalation-protocol.md references/escalation-protocol.md
rmdir skills/auto-academic/references
```

- [ ] **Step 2: Commit**

```bash
git add references/venue-profiles.md references/experiment-patterns.md references/convergence-criteria.md references/escalation-protocol.md
git add -u skills/auto-academic/references/
git commit -m "refactor: move auto-academic references to top-level references/"
```

---

## Phase 5: Consolidate Templates and Examples

### Task 14: Copy all upstream templates (20 files)

**Files:**
- Create: 6 deep-research templates
- Create: 10 academic-paper templates
- Create: 3 academic-paper-reviewer templates
- Create: 1 academic-pipeline template

- [ ] **Step 1: Copy deep-research templates**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/templates/research_brief_template.md templates/research-brief.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/templates/literature_matrix_template.md templates/literature-matrix.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/templates/evidence_assessment_template.md templates/evidence-assessment.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/templates/preregistration_template.md templates/preregistration.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/templates/prisma_protocol_template.md templates/prisma-protocol.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/templates/prisma_report_template.md templates/prisma-report.md
```

- [ ] **Step 2: Copy academic-paper templates**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/bilingual_abstract_template.md templates/bilingual-abstract.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/case_study_template.md templates/case-study.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/conference_paper_template.md templates/conference-paper.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/credit_statement_template.md templates/credit-statement.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/funding_statement_template.md templates/funding-statement.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/imrad_template.md templates/imrad.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/literature_review_template.md templates/literature-review.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/policy_brief_template.md templates/policy-brief.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/revision_tracking_template.md templates/revision-tracking.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/templates/theoretical_paper_template.md templates/theoretical-paper.md
```

- [ ] **Step 3: Copy reviewer + pipeline templates**

```bash
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/templates/editorial_decision_template.md templates/editorial-decision.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/templates/peer_review_report_template.md templates/peer-review-report.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/templates/revision_response_template.md templates/revision-response.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/templates/pipeline_status_template.md templates/pipeline-status.md
```

- [ ] **Step 4: Commit**

```bash
git add templates/
git commit -m "feat: integrate upstream templates (20 files, kebab-case renamed)"
```

---

### Task 15: Migrate existing auto-academic templates

Move templates from `skills/auto-academic/templates/` to top-level `templates/`.

**Files:**
- Move: `skills/auto-academic/templates/pipeline-state.json` → `templates/pipeline-state.json`
- Move: `skills/auto-academic/templates/experiment-spec.md` → `templates/experiment-spec.md`
- Move: `skills/auto-academic/templates/claim-evaluation.md` → `templates/claim-evaluation.md`
- Move: `skills/auto-academic/templates/process-log.md` → `templates/process-log.md`
- Move: `skills/auto-academic/templates/results-tsv-header.tsv` → `templates/results-tsv-header.tsv`

- [ ] **Step 1: Move templates**

```bash
mv skills/auto-academic/templates/pipeline-state.json templates/pipeline-state.json
mv skills/auto-academic/templates/experiment-spec.md templates/experiment-spec.md
mv skills/auto-academic/templates/claim-evaluation.md templates/claim-evaluation.md
mv skills/auto-academic/templates/process-log.md templates/process-log.md
mv skills/auto-academic/templates/results-tsv-header.tsv templates/results-tsv-header.tsv
rmdir skills/auto-academic/templates
```

- [ ] **Step 2: Commit**

```bash
git add templates/pipeline-state.json templates/experiment-spec.md templates/claim-evaluation.md templates/process-log.md templates/results-tsv-header.tsv
git add -u skills/auto-academic/templates/
git commit -m "refactor: move auto-academic templates to top-level templates/"
```

---

### Task 16: Copy examples and shared utilities

**Files:**
- Create: `examples/deep-research/` (7 files)
- Create: `examples/academic-paper/` (6 files)
- Create: `examples/academic-paper-reviewer/` (2 files)
- Create: `examples/academic-pipeline/` (3 files + showcase/)
- Create: `shared/handoff-schemas.md`
- Create: `shared/style-calibration-protocol.md`
- Move: `skills/auto-academic/examples/` → `examples/auto-academic/`

- [ ] **Step 1: Copy deep-research examples**

```bash
mkdir -p examples/deep-research
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/examples/exploratory_research.md examples/deep-research/exploratory-research.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/examples/fact_check_mode.md examples/deep-research/fact-check-mode.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/examples/handoff_to_paper.md examples/deep-research/handoff-to-paper.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/examples/policy_analysis.md examples/deep-research/policy-analysis.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/examples/review_mode.md examples/deep-research/review-mode.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/examples/socratic_guided_research.md examples/deep-research/socratic-guided-research.md
cp /Users/tom/Project/auto-academic/academic-research-skills/deep-research/examples/systematic_review.md examples/deep-research/systematic-review.md
```

- [ ] **Step 2: Copy academic-paper examples**

```bash
mkdir -p examples/academic-paper
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/examples/chinese_paper_example.md examples/academic-paper/chinese-paper.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/examples/imrad_hei_example.md examples/academic-paper/imrad-hei.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/examples/literature_review_example.md examples/academic-paper/literature-review.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/examples/plan_mode_guided_writing.md examples/academic-paper/plan-mode-guided-writing.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/examples/revision_mode_example.md examples/academic-paper/revision-mode.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper/examples/revision_recovery_example.md examples/academic-paper/revision-recovery.md
```

- [ ] **Step 3: Copy reviewer + pipeline examples + showcase**

```bash
mkdir -p examples/academic-paper-reviewer examples/academic-pipeline
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/examples/hei_paper_review_example.md examples/academic-paper-reviewer/hei-paper-review.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-paper-reviewer/examples/interdisciplinary_review_example.md examples/academic-paper-reviewer/interdisciplinary-review.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/examples/full_pipeline_example.md examples/academic-pipeline/full-pipeline.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/examples/integrity_failure_recovery.md examples/academic-pipeline/integrity-failure-recovery.md
cp /Users/tom/Project/auto-academic/academic-research-skills/academic-pipeline/examples/mid_entry_example.md examples/academic-pipeline/mid-entry.md
cp -r /Users/tom/Project/auto-academic/academic-research-skills/examples/showcase examples/academic-pipeline/showcase
```

- [ ] **Step 4: Copy shared utilities**

```bash
mkdir -p shared
cp /Users/tom/Project/auto-academic/academic-research-skills/shared/handoff_schemas.md shared/handoff-schemas.md
cp /Users/tom/Project/auto-academic/academic-research-skills/shared/style_calibration_protocol.md shared/style-calibration-protocol.md
```

- [ ] **Step 5: Move existing auto-academic examples**

```bash
mkdir -p examples/auto-academic
mv skills/auto-academic/examples/education-research.md examples/auto-academic/education-research.md
mv skills/auto-academic/examples/cs-ml-research.md examples/auto-academic/cs-ml-research.md
mv skills/auto-academic/examples/mixed-methods.md examples/auto-academic/mixed-methods.md
rmdir skills/auto-academic/examples
```

- [ ] **Step 6: Commit**

```bash
git add examples/ shared/
git add -u skills/auto-academic/examples/
git commit -m "feat: integrate examples (26 files + showcase) and shared utilities"
```

---

## Phase 6: Create program-spine.md Template

### Task 17: Create the program-spine.md template

**Files:**
- Create: `templates/program-spine.md`

- [ ] **Step 1: Create program-spine.md**

This is the fixed spine template from the program.md design spec. The experiment-planner fills in the freeform sections per hypothesis.

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add templates/program-spine.md
git commit -m "feat: add program-spine.md template for autonomous experiment loops"
```

---

## Phase 7: Update Internal References

### Task 18: Update all SKILL.md files — path references

Every SKILL.md that references agents, references, or templates needs paths updated from upstream nested structure to consolidated top-level structure.

**Files:**
- Modify: `skills/deep-research/SKILL.md`
- Modify: `skills/academic-paper/SKILL.md`
- Modify: `skills/academic-paper-reviewer/SKILL.md`
- Modify: `skills/academic-pipeline/SKILL.md`

- [ ] **Step 1: Update deep-research/SKILL.md**

Search and replace all path patterns:
- `agents/research_question_agent` → `${CLAUDE_PLUGIN_ROOT}/agents/research-question`
- `agents/bibliography_agent` → `${CLAUDE_PLUGIN_ROOT}/agents/bibliography`
- (repeat for all 13 agents)
- `references/apa7_style_guide.md` → `${CLAUDE_PLUGIN_ROOT}/references/apa7-guide.md`
- `references/failure_paths.md` → `${CLAUDE_PLUGIN_ROOT}/references/failure-paths.md`
- `references/mode_selection_guide.md` → `${CLAUDE_PLUGIN_ROOT}/references/mode-selection-guide.md`
- (repeat for all other references — underscore_case to kebab-case, add `${CLAUDE_PLUGIN_ROOT}/references/` prefix)
- `templates/research_brief_template.md` → `${CLAUDE_PLUGIN_ROOT}/templates/research-brief.md`
- (repeat for all templates)

- [ ] **Step 2: Update academic-paper/SKILL.md**

Same pattern — update all agent, reference, and template paths to `${CLAUDE_PLUGIN_ROOT}/` prefixed kebab-case paths.

- [ ] **Step 3: Update academic-paper-reviewer/SKILL.md**

Same pattern.

- [ ] **Step 4: Update academic-pipeline/SKILL.md**

Same pattern.

- [ ] **Step 5: Commit**

```bash
git add skills/deep-research/SKILL.md skills/academic-paper/SKILL.md skills/academic-paper-reviewer/SKILL.md skills/academic-pipeline/SKILL.md
git commit -m "refactor: update academic skill paths to consolidated top-level structure"
```

---

### Task 19: Update all agent files — path references

Every agent that references other agents or reference files needs updated paths.

**Files:**
- Modify: All 40 agent files in `agents/`

- [ ] **Step 1: Batch update agent internal references**

For each agent file in `agents/`:
1. Replace any relative paths to references (e.g., `../references/`) with `${CLAUDE_PLUGIN_ROOT}/references/`
2. Replace any agent cross-references with kebab-case names
3. Replace underscore_case filenames with kebab-case

Use search-and-replace across all files in `agents/`:
- `_agent.md` → `.md` (in agent cross-references)
- `_agent` → `` (in inline text references)
- Underscore paths → kebab-case paths

- [ ] **Step 2: Verify no broken references**

```bash
grep -r "references/" agents/ | grep -v "CLAUDE_PLUGIN_ROOT" | head -20
grep -r "_agent" agents/ | head -20
```

Both should return empty or only valid inline text (not file paths).

- [ ] **Step 3: Commit**

```bash
git add agents/
git commit -m "refactor: update agent path references to consolidated structure"
```

---

### Task 20: Update auto-academic SKILL.md — path references

The main orchestrator SKILL.md needs updated references since templates/references moved.

**Files:**
- Modify: `skills/auto-academic/SKILL.md`

- [ ] **Step 1: Update references paths**

Replace:
- `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/references/convergence-criteria.md` → `${CLAUDE_PLUGIN_ROOT}/references/convergence-criteria.md`
- `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/references/escalation-protocol.md` → `${CLAUDE_PLUGIN_ROOT}/references/escalation-protocol.md`
- `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/references/experiment-patterns.md` → `${CLAUDE_PLUGIN_ROOT}/references/experiment-patterns.md`
- `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/references/venue-profiles.md` → `${CLAUDE_PLUGIN_ROOT}/references/venue-profiles.md`

- [ ] **Step 2: Update template paths**

Replace:
- `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/templates/` → `${CLAUDE_PLUGIN_ROOT}/templates/`

- [ ] **Step 3: Update skill invocation names**

Replace:
- `deep-research` (as skill invocation) → `auto-academic:deep-research`
- `academic-paper` (as skill invocation) → `auto-academic:academic-paper`
- `academic-paper-reviewer` (as skill invocation) → `auto-academic:academic-paper-reviewer`
- `superpowers:writing-plans` → `auto-academic:writing-plans`
- `superpowers:executing-plans` → `auto-academic:executing-plans`
- `superpowers:dispatching-parallel-agents` → `auto-academic:dispatching-parallel-agents`

- [ ] **Step 4: Update dependency declaration**

Replace the dependency section that says plugins must be installed with:
```
All required skills are bundled within this plugin. No external plugin dependencies.
```

- [ ] **Step 5: Commit**

```bash
git add skills/auto-academic/SKILL.md
git commit -m "refactor: update auto-academic SKILL.md to use consolidated paths and namespaced skills"
```

---

### Task 21: Update auto-academic agent files — path references

The 4 existing auto-academic agents reference files in the old nested structure.

**Files:**
- Modify: `agents/experiment-planner.md`
- Modify: `agents/experiment-evaluator.md`
- Modify: `agents/convergence-judge.md`
- Modify: `agents/escalation-analyst.md`

- [ ] **Step 1: Update experiment-planner.md**

Replace:
- `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/references/` → `${CLAUDE_PLUGIN_ROOT}/references/`
- `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/templates/` → `${CLAUDE_PLUGIN_ROOT}/templates/`

- [ ] **Step 2: Update experiment-evaluator.md**

Same pattern.

- [ ] **Step 3: Update convergence-judge.md**

Same pattern.

- [ ] **Step 4: Update escalation-analyst.md**

Same pattern.

- [ ] **Step 5: Commit**

```bash
git add agents/experiment-planner.md agents/experiment-evaluator.md agents/convergence-judge.md agents/escalation-analyst.md
git commit -m "refactor: update auto-academic agents to use top-level references/templates paths"
```

---

## Phase 8: Merge Orchestrator

### Task 22: Merge auto-academic + academic-pipeline into unified 8-stage SKILL.md

This is the most complex task. The merged SKILL.md combines auto-academic's dual-loop with academic-pipeline's integrity verification.

**Files:**
- Modify: `skills/auto-academic/SKILL.md`

- [ ] **Step 1: Read both source files**

Read:
- `skills/auto-academic/SKILL.md` (current)
- `skills/academic-pipeline/SKILL.md` (just integrated)

Identify the sections to merge:
- From academic-pipeline: integrity verification stages, adaptive checkpoint system, material passport, process summary generation, state tracker integration
- From auto-academic: keep dual-loop architecture, autonomous experiment loop, convergence/escalation logic

- [ ] **Step 2: Add Stage 0 enhancements (from academic-pipeline)**

Add to the INIT stage:
- Material passport check for mid-pipeline resume capability
- State tracker agent initialization

- [ ] **Step 3: Add Stage 4 — INTEGRITY (from academic-pipeline)**

Insert new stage between WRITE and REVIEW:
- Invoke `integrity-verification` agent
- 5-phase check: references, citations, data, statistics, claims
- CHECKPOINT: MANDATORY — must pass 100%
- Retry logic: max 3 attempts, targeted fix per attempt

- [ ] **Step 4: Add Stage 6 — FINAL INTEGRITY (from academic-pipeline)**

Insert new stage between REVIEW and FINALIZE:
- Post-revision integrity re-verification
- CHECKPOINT: MANDATORY

- [ ] **Step 5: Add adaptive checkpoint system (from academic-pipeline)**

Update all stages with checkpoint types:
- Stage 1 (RESEARCH): MANDATORY
- Stage 2 (EXPERIMENT): none (autonomous)
- Stage 3 (WRITE): ADAPTIVE
- Stage 4 (INTEGRITY): MANDATORY
- Stage 5 (REVIEW): ADAPTIVE (FULL first, SLIM after)
- Stage 6 (FINAL INTEGRITY): MANDATORY
- Stage 7 (FINALIZE): MANDATORY

- [ ] **Step 6: Add process summary generation to FINALIZE (from academic-pipeline)**

Add bilingual process summary generation step to Stage 7.

- [ ] **Step 7: Update stage numbering and transitions**

Renumber all stages: INIT(0), RESEARCH(1), EXPERIMENT(2), WRITE(3), INTEGRITY(4), REVIEW(5), FINAL_INTEGRITY(6), FINALIZE(7).

Update all transition logic (stage exit conditions, loop re-entry points).

- [ ] **Step 8: Commit**

```bash
git add skills/auto-academic/SKILL.md
git commit -m "feat: merge academic-pipeline integrity verification into unified 8-stage pipeline"
```

---

### Task 23: Update pipeline-state.json template for 8 stages

**Files:**
- Modify: `templates/pipeline-state.json`

- [ ] **Step 1: Update pipeline-state.json**

Add the new stages:

```json
{
  "version": "2.0.0",
  "topic": "",
  "venue": "",
  "current_stage": "INIT",
  "material_passport": {
    "entry_point": "INIT",
    "provenance": []
  },
  "stages": {
    "INIT": { "status": "pending" },
    "RESEARCH": { "status": "pending", "mode": "full" },
    "EXPERIMENT": {
      "status": "pending",
      "inner_loop_iteration": 0,
      "hypotheses": [],
      "experiment_runs": [],
      "programs": []
    },
    "WRITE": { "status": "pending", "draft_version": 0 },
    "INTEGRITY": {
      "status": "pending",
      "verification_pass": false,
      "retry_count": 0,
      "max_retries": 3,
      "phases_passed": []
    },
    "REVIEW": {
      "status": "pending",
      "outer_loop_round": 0,
      "max_outer_rounds": 5,
      "escalation_count": 0,
      "decisions": [],
      "checkpoint_mode": "FULL"
    },
    "FINAL_INTEGRITY": {
      "status": "pending",
      "verification_pass": false
    },
    "FINALIZE": { "status": "pending" }
  }
}
```

- [ ] **Step 2: Commit**

```bash
git add templates/pipeline-state.json
git commit -m "feat: update pipeline-state.json for 8-stage unified pipeline"
```

---

## Phase 9: Update experiment-planner for program.md generation

### Task 24: Update experiment-planner agent to generate program.md

**Files:**
- Modify: `agents/experiment-planner.md`

- [ ] **Step 1: Add program.md generation responsibility**

Add a new section to the experiment-planner agent instructions:

After generating the experiment spec, the planner must also:
1. Read `${CLAUDE_PLUGIN_ROOT}/templates/program-spine.md`
2. Fill in all `{{FREEFORM}}` sections based on the experiment spec and hypothesis
3. Select appropriate experiment pattern from `${CLAUDE_PLUGIN_ROOT}/references/experiment-patterns.md`
4. Write the completed `program.md` to `experiments/programs/{hypothesis_id}/program.md`
5. Write the initial `main.py` to `experiments/programs/{hypothesis_id}/main.py`

The planner uses the experiment pattern to determine:
- Tier 1 tunable parameters (specific to experiment type)
- Tier 2 code change boundaries
- Evaluation criteria (what "better" means for this experiment type)
- Output format (which metrics to print)
- Read-only files and dependencies

- [ ] **Step 2: Add program.md examples per experiment pattern**

Update `references/experiment-patterns.md` to include a program.md example snippet for each of the 7 patterns, showing what the freeform Scope and Evaluation sections look like for:
1. Survey/questionnaire analysis
2. Text/content analysis
3. ML classification/regression
4. Simulation/agent-based modeling
5. Statistical comparison/meta-analysis
6. Network/graph analysis
7. Natural experiment/quasi-experimental

- [ ] **Step 3: Commit**

```bash
git add agents/experiment-planner.md references/experiment-patterns.md
git commit -m "feat: update experiment-planner to generate program.md per hypothesis"
```

---

## Phase 10: Update CLAUDE.md and Final Cleanup

### Task 25: Update CLAUDE.md

**Files:**
- Modify: `/Users/tom/Project/auto-academic/CLAUDE.md`

- [ ] **Step 1: Update architecture section**

Replace the current architecture description with the unified plugin structure. Remove references to external plugin dependencies. Update the file layout diagram to show the consolidated structure.

- [ ] **Step 2: Update installation section**

Remove the requirement to install academic-research-skills and superpowers as separate plugins. Update to: "Clone or copy the `auto-academic/` directory to `~/.claude/plugins/auto-academic/`."

- [ ] **Step 3: Update pipeline flow diagram**

Replace the 6-stage diagram with the 8-stage merged pipeline:
```
INIT → RESEARCH → EXPERIMENT → WRITE → INTEGRITY → REVIEW → FINAL_INTEGRITY → FINALIZE
         ↑                                           |
         └── inner loop (autonomous) ────────────────┘
```

- [ ] **Step 4: Update plugin dependencies section**

Replace with: "All required skills are bundled within this plugin. No external plugin dependencies."

- [ ] **Step 5: Commit**

```bash
git add /Users/tom/Project/auto-academic/CLAUDE.md
git commit -m "docs: update CLAUDE.md for unified self-contained plugin"
```

---

### Task 26: Clean up old nested structure

**Files:**
- Delete: `skills/auto-academic/references/` (already moved)
- Delete: `skills/auto-academic/templates/` (already moved)
- Delete: `skills/auto-academic/examples/` (already moved)

- [ ] **Step 1: Verify directories are empty**

```bash
ls skills/auto-academic/references/ 2>/dev/null && echo "NOT EMPTY - DO NOT DELETE" || echo "OK: empty or gone"
ls skills/auto-academic/templates/ 2>/dev/null && echo "NOT EMPTY - DO NOT DELETE" || echo "OK: empty or gone"
ls skills/auto-academic/examples/ 2>/dev/null && echo "NOT EMPTY - DO NOT DELETE" || echo "OK: empty or gone"
```

All should report "OK: empty or gone" (moved in earlier tasks).

- [ ] **Step 2: Verify the plugin structure is complete**

```bash
echo "=== Skills ===" && ls skills/
echo "=== Agents ===" && ls agents/ | wc -l
echo "=== References ===" && ls references/ | wc -l
echo "=== Templates ===" && ls templates/ | wc -l
echo "=== Examples ===" && ls examples/
echo "=== Hooks ===" && ls hooks/
echo "=== Shared ===" && ls shared/
```

Expected:
- Skills: 19 directories
- Agents: 40 files
- References: ~48 files
- Templates: ~27 files
- Examples: 5 directories
- Hooks: 3 files
- Shared: 2 files

- [ ] **Step 3: Commit final cleanup**

```bash
git add -A
git commit -m "chore: clean up old nested structure, verify unified plugin layout"
```
