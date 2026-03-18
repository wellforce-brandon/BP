---
concern: claude-config
tech: [claude-code]
priority: foundational
source-repo: Janki
applies-to: [all]
---
# Hierarchical CLAUDE.md Structure

## PATTERN
Use a layered CLAUDE.md architecture that loads context top-down: root level for project-wide rules, subfolder level for domain-specific conventions, and `.claude/rules/*.md` for path-scoped conditional instructions.

- **Root `CLAUDE.md`** -- Project-wide rules, tech stack, coding standards, available skills/agents, numbered RULES. Keep under 150-200 lines.
- **Subfolder `CLAUDE.md`** -- Only when a subfolder has distinct conventions (e.g., `frontend/CLAUDE.md` for UI rules, `backend/CLAUDE.md` for API rules). Only loads when working in that path.
- **`.claude/rules/*.md`** -- Conditional instructions with `paths:` frontmatter. Only load when working with matching file patterns.

## WHY
A single monolithic CLAUDE.md grows unmanageable and wastes context on irrelevant instructions. The hierarchical approach ensures Claude only loads what's relevant to the current task, keeping adherence high and context usage low.

## EXAMPLE
From Janki (`CLAUDE.md`):
```markdown
## Hierarchical CLAUDE.md Architecture

CLAUDE.md files load top-down: root user level, then project level, then subfolder level.
Only relevant files load -- a frontend task never loads the backend CLAUDE.md.

- Root `CLAUDE.md` -- Project-wide rules, stack, global conventions (this file).
- Subfolder `CLAUDE.md` -- Only when subfolder has distinct conventions.
- `.claude/rules/*.md` -- Conditional instructions with `paths:` frontmatter.
- Keep each file focused. Prune after every model update.
- Do NOT bloat CLAUDE.md with generic advice the model already knows.
```

## CHECK
How to verify if a repo already follows this:
- [ ] Root `CLAUDE.md` exists and is under 200 lines
- [ ] Root `CLAUDE.md` includes tech stack, coding standards, and workflow sections
- [ ] No subfolder CLAUDE.md files exist that duplicate root-level content
- [ ] If repo has distinct domains (frontend/backend), subfolder CLAUDE.md files exist for each

## IMPLEMENT
Steps to adopt this in a repo that doesn't have it:
1. Create or refactor root `CLAUDE.md` with: project overview, tech stack, coding standards, available skills/agents, workflow rules, numbered RULES
2. Move domain-specific content to subfolder `CLAUDE.md` files where appropriate
3. Extract path-conditional rules to `.claude/rules/*.md` with `paths:` frontmatter
4. Prune root CLAUDE.md to under 200 lines -- remove anything the model already knows natively

## NOTES
- Keep each file focused on what Claude needs to know that it wouldn't figure out on its own
- Prune after every major model update -- newer models handle more things natively
- The 150-200 line limit is practical: longer files see declining adherence
