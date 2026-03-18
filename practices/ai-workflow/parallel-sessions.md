---
concern: ai-workflow
tech: [claude-code, git]
priority: recommended
source-repo: external
applies-to: [all]
---
# Parallel Sessions and Fan-Out Patterns

## PATTERN
Three patterns for parallelizing AI-assisted work:

1. **Writer/Reviewer** -- One session implements, a fresh session reviews. The fresh session has no bias toward code it just wrote, catching more issues.
2. **Fan-Out** -- For batch operations (migrations, refactors across many files), use `claude -p "Migrate $file..." --allowedTools "Edit,Bash"` in a loop to process files in parallel.
3. **Git Worktrees** -- Use `git worktree add` to create parallel working directories. Each AI session works in its own worktree, avoiding checkout conflicts. Claude Code supports `isolation: worktree` in agent definitions.

## WHY
Linear AI work (one task at a time, one session) doesn't scale for large codebases. Parallel sessions multiply throughput for independent tasks. The writer/reviewer pattern specifically addresses the bias problem where an AI reviewing its own output is less likely to catch errors than a fresh session.

## EXAMPLE
Fan-out migration:
```bash
# Generate file list, process each in parallel
files=$(find src -name "*.ts" -path "*/api/*")
for file in $files; do
  claude -p "Migrate $file from Express to Fastify. Follow
    the patterns in src/api/users.ts as reference." \
    --allowedTools "Edit,Read" &
done
wait
```

Writer/Reviewer in CLAUDE.md:
```markdown
## Quality Workflow
For important changes:
1. Implement in the current session
2. Commit the implementation
3. Start a fresh session: `claude --resume`
4. Ask the fresh session to review the last commit
```

Git worktree agent:
```yaml
name: feature-agent
isolation: worktree
model: sonnet
```

## CHECK
- [ ] Team knows about `claude -p` for batch processing
- [ ] Writer/reviewer pattern is documented for critical changes
- [ ] Git worktree usage is understood for parallel agent work

## IMPLEMENT
1. Document the writer/reviewer pattern in CLAUDE.md
2. For batch operations, create a shell script using `claude -p` in a loop
3. Use `isolation: worktree` in agent definitions for independent work
4. Limit to 3-4 concurrent tasks to avoid resource contention

## NOTES
- Source: Anthropic official best practices, shanraisshan/claude-code-best-practice
- Subagents within a session are different from parallel sessions -- subagents share the session's context budget
- `claude -p` runs a single prompt and exits -- useful for scripted batch operations
- Always review fan-out results before committing -- parallel operations can create conflicts
