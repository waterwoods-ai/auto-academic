# auto-academic

A Claude Code plugin that runs a fully autonomous academic research pipeline. Give it a topic and venue — it handles research, experiments, paper writing, peer review, and revision without stopping until the paper is accepted or safety caps are hit.

## How it works

```
INIT → RESEARCH → EXPERIMENT → WRITE → REVIEW → FINALIZE
         ↑                        |
         └── inner loop ──────────┘  (research + experiments, keep/discard)
                                  |
                           REVIEW ←→ REVISE  (outer loop, tiered escalation)
```

**Inner loop**: Deep research + computational experiments with autoresearch-style keep/discard decisions. A convergence judge exits the loop when all hypotheses have evidence.

**Outer loop**: Peer review + revision. If the reviewer issues Major/Reject at round 3, an escalation analyst re-enters the inner loop to address methodology concerns. Force-finalizes at round 5.

**Safety caps**: Max 10 experiment runs (inner), 5 review rounds (outer), 1 escalation per pipeline run.

## Requirements

This plugin depends on two other Claude Code plugins:

- **[academic-research-skills](https://github.com/Cheng-I-Wu/academic-research-skills)** — provides `deep-research`, `academic-paper`, `academic-paper-reviewer` skills
- **[superpowers](https://github.com/gstack-com/claude-code-power-pack)** — provides `writing-plans`, `executing-plans`, `dispatching-parallel-agents` skills

Install both before using auto-academic.

## Installation

1. Install the required plugins above.
2. Copy the `auto-academic/` directory (excluding `.git/`) to your Claude Code plugins directory:

```bash
cp -r auto-academic/ ~/.claude/plugins/auto-academic/
```

The plugin uses Claude Code's auto-discovery — no manual registration needed.

## Usage

```
/auto-academic "Impact of LLMs on undergraduate writing skills" --venue "Computers & Education"
```

**Arguments:**
- `topic` (required) — the research question or area
- `--venue` (optional) — target publication venue. Defaults to APA 7.0, Tier 3 standards if omitted.

The pipeline adapts rigor to the venue tier (Tier 1 = NeurIPS/ICML, Tier 5 = workshop).

## Plugin structure

```
auto-academic/
├── .claude-plugin/plugin.json       # Plugin manifest
├── commands/
│   └── auto-academic.md             # /auto-academic slash command
├── agents/
│   ├── experiment-planner.md        # Designs computational experiments
│   ├── experiment-evaluator.md      # Assesses results (keep/discard/refine)
│   ├── convergence-judge.md         # Decides when loops should exit
│   └── escalation-analyst.md        # Handles review escalation
└── skills/
    └── auto-academic/
        ├── SKILL.md                 # Main orchestrator
        ├── references/              # Canonical rules (single source of truth)
        ├── templates/               # State & spec templates
        └── examples/                # Pipeline walkthrough demos
```

## Examples

The `skills/auto-academic/examples/` directory contains three pipeline traces:

- **education-research.md** — Tier 3 SSCI journal, 3 review rounds
- **cs-ml-research.md** — Tier 1 conference (ACL), baseline + ablations
- **mixed-methods.md** — Tier 3 with escalation workflow demonstration

## License

MIT
