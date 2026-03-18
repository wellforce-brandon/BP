---
concern: context-management
tech: [claude-code]
priority: recommended
source-repo: 60k-mono
applies-to: [all]
---
# Shared Agent Memory for Cross-Session Knowledge

## PATTERN
Use `.claude/agent-memory/` as a version-controlled, team-shared knowledge base that persists across sessions. Three standard files: `patterns.md` (confirmed codebase patterns), `decisions.md` (architectural decisions with rationale), `debugging.md` (solutions to recurring problems). First 200 lines of MEMORY.md are auto-injected into agent system prompts.

## WHY
Each Claude Code session starts fresh. Without persistent memory, teams re-discover the same patterns and re-debug the same issues. Agent memory solves this by capturing evolving knowledge that can't be derived from code alone -- the "why" behind decisions, workarounds for known issues, and confirmed patterns that aren't obvious from reading the code.

## EXAMPLE
From 60k-mono (`.claude/agent-memory/README.md`):
```markdown
## Scopes
- **project** (`.claude/agent-memory/`): version-controlled, team-shared
- **local** (`.claude/agent-memory-local/`): git-ignored, personal
- **user** (`~/.claude/agent-memory/`): cross-project, personal

## Files
| File | Purpose |
|------|---------|
| patterns.md | Confirmed codebase patterns and conventions |
| decisions.md | Architectural decisions and their rationale |
| debugging.md | Solutions to recurring problems |
```

Example `decisions.md` entry:
```markdown
## Drizzle 0.45.x (not v1 beta)
- v1.0.0-beta has breaking migration structure changes
- Stay on 0.45.x until deliberate upgrade window planned
- Caret range won't auto-resolve to v1 beta (tagged `beta`)
```

## CHECK
- [ ] `.claude/agent-memory/` directory exists
- [ ] At least `patterns.md` and `decisions.md` exist
- [ ] CLAUDE.md references agent memory or explains cross-session knowledge sharing
- [ ] `.claude/agent-memory-local/` is in `.gitignore`

## IMPLEMENT
1. Create `.claude/agent-memory/` directory
2. Create `README.md` explaining the three scopes and file purposes
3. Create `patterns.md`, `decisions.md`, `debugging.md` (can be empty initially)
4. Add `.claude/agent-memory-local/` to `.gitignore`
5. Document in CLAUDE.md that sessions should capture discoveries to agent memory

## NOTES
- Keep MEMORY.md under 200 lines (auto-injected into prompts)
- Never overwrite, only append or update
- Remove stale entries when patterns change
- debugging.md prevents repeating failed approaches across sessions
- Source: 60k-mono, Janki, awesome-copilot instructions pattern
