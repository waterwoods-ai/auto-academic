# auto-academic

A Claude Code plugin that runs a fully autonomous academic research pipeline. Give it a topic and venue — it handles research, experiments, paper writing, peer review, and revision without stopping until the paper is accepted or safety caps are hit.

## How it works

```
INIT → RESEARCH → EXPERIMENT → WRITE → INTEGRITY → REVIEW → FINAL_INTEGRITY → FINALIZE
         ↑                                           |
         └── inner loop (autonomous) ────────────────┘
                                                     |
                                              REVIEW ←→ REVISE (outer loop)
```

**Inner loop**: Deep research + computational experiments with autoresearch-style keep/discard decisions. A convergence judge exits the loop when all hypotheses have evidence.

**Outer loop**: Peer review + revision. If the reviewer issues Major/Reject at round 3, an escalation analyst re-enters the inner loop to address methodology concerns. Force-finalizes at round 5.

**Safety caps**: Max 10 experiment runs (inner), 5 review rounds (outer), 1 escalation per pipeline run.

## Requirements

This is a self-contained plugin. All required skills, agents, references, and templates are bundled. No external plugin dependencies.

See the `NOTICE` file for upstream attribution (academic-research-skills, superpowers).

## Installation

Copy the `auto-academic/` directory (excluding `.git/`) to your Claude Code plugins directory:

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
├── .claude-plugin/plugin.json
├── commands/auto-academic.md
├── hooks/
│   ├── hooks.json
│   ├── run-hook.cmd
│   └── session-start
├── skills/                    # 19 skills (auto-academic + academic + superpowers)
├── agents/                    # 40 agents consolidated
├── references/                # ~48 references (deduplicated)
├── templates/                 # ~27 templates
├── examples/                  # Examples organized by skill
├── shared/                    # Inter-skill utilities
├── autoresearch/              # Reference implementation (upstream)
├── NOTICE                     # Upstream attribution
└── LICENSE
```

## Examples

The `examples/` directory contains pipeline traces organized by skill:

- **auto-academic/education-research.md** — Tier 3 SSCI journal, 3 review rounds
- **auto-academic/cs-ml-research.md** — Tier 1 conference (ACL), baseline + ablations
- **auto-academic/mixed-methods.md** — Tier 3 with escalation workflow demonstration

## License

MIT
