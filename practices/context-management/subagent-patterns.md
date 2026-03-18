---
concern: context-management
tech: [claude-code]
priority: recommended
source-repo: Janki
applies-to: [all]
---
# Subagent Usage Patterns

## PATTERN
Aggressively offload to subagents for: online research, doc fetching, log analysis, codebase exploration. Always include a "why" in every subagent prompt -- not just what to find, but why you need it. When torn between approaches, spin up parallel Explore subagents for each option. Subagents are resumable -- continue a specific subagent to extend its research.

## WHY
Subagents protect the main context window from pollution. Research that loads 10 files into context to find one answer wastes 90% of that context. A subagent does the research, returns only the answer, and the main session stays lean. Parallel subagents are especially powerful for comparing approaches without committing to one.

## EXAMPLE
From Janki CLAUDE.md:
```markdown
## Subagent Usage

Always and aggressively offload to subagents: online research,
doc fetching, log analysis, codebase exploration. This keeps the
main context narrow.

- **Always include a "why"** in every subagent prompt. Not just
  what to find, but why you need it. "How auth works for rate
  limiting because we're improving rate limiting" beats "how
  auth works."
- **Parallel exploration:** When torn between approaches, spin up
  parallel Explore subagents for each, pass results back, let
  the main session decide.
- **Subagents are resumable.** You can resume a specific subagent
  to continue its research.
```

Good prompt: "Search for Vitest monorepo config patterns. WHY: We need to set up per-package test configs that share a base config across 8 packages."

Bad prompt: "Search for Vitest config."

## CHECK
- [ ] CLAUDE.md mentions subagent usage with guidance on when to offload
- [ ] Subagent guidance includes the "always include a why" rule
- [ ] Parallel exploration pattern is documented

## IMPLEMENT
1. Add subagent usage section to CLAUDE.md
2. Include the "why" rule and parallel exploration pattern
3. List common offload tasks: research, doc fetching, log analysis, codebase exploration

## NOTES
- The Explore agent type is optimized for codebase searches
- Use `subagent_type: "Explore"` for file/code searches, `general-purpose` for multi-step research
- Subagent results are not visible to the user -- summarize key findings in the main session
- Source: Janki CLAUDE.md, awesome-claude-code patterns
