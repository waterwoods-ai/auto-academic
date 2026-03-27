---
name: auto-academic
description: "Run the fully autonomous academic research pipeline: research -> experiments -> paper -> review -> revision loop"
arguments:
  - name: topic
    description: "Research topic (required)"
    required: true
  - name: venue
    description: "Target publication venue (optional, defaults to APA 7.0 Tier 3)"
    required: false
---

Load and follow the auto-academic SKILL.md to run the fully autonomous academic research pipeline.

**Topic:** $ARGUMENTS.topic
**Venue:** $ARGUMENTS.venue

Invoke the skill referenced at `${CLAUDE_PLUGIN_ROOT}/skills/auto-academic/SKILL.md` and execute it with the provided topic and venue.
