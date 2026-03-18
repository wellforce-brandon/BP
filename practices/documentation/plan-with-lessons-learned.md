---
concern: documentation
tech: [claude-code]
priority: foundational
source-repo: 60k-mono
applies-to: [all]
---
# Plans Must End with Lessons Learned

## PATTERN
Every plan file (in `tasks/` or plan mode) MUST end with a "Lessons Learned / Gotchas" section. After implementation, discoveries are routed to LL-G via `/add-lesson` rather than staying in local files.

## WHY
Plans are where you discover the unexpected -- API quirks, framework limitations, config gotchas. Without a structured place to capture these, they're lost when the session ends. The Lessons Learned section creates a deliberate checkpoint. Routing discoveries to LL-G ensures they benefit all future work across all repos, not just the current one.

## EXAMPLE
From 60k-mono (`CLAUDE.md`):
```markdown
**Plans**: Every plan MUST end with a "Lessons Learned / Gotchas"
section. New generalizable lessons go to LL-G, not local files.
```

Plan template:
```markdown
# [Plan Title]

## Context
[What problem we're solving and why]

## Steps
1. ...
2. ...

## Files
- [Key files involved]

## Lessons Learned / Gotchas
After implementation, capture here:
- [ ] New patterns discovered -- route to LL-G
- [ ] Gotchas encountered -- add to LL-G with PROBLEM/WRONG/RIGHT/NOTES
- [ ] Workflow improvements -- update CLAUDE.md or memory
```

## CHECK
How to verify if a repo already follows this:
- [ ] CLAUDE.md mentions the Lessons Learned requirement for plans
- [ ] Existing plan files in `tasks/` include a Lessons Learned section
- [ ] CLAUDE.md references LL-G contribution workflow

## IMPLEMENT
1. Add to CLAUDE.md: "Every plan MUST end with a Lessons Learned / Gotchas section"
2. Add to CLAUDE.md: "New generalizable lessons go to LL-G, not local files"
3. Include the plan template in `.claude/references/` or CLAUDE.md
4. Review existing plans and backfill Lessons Learned sections if missing

## NOTES
- "Lessons stored locally stay local. Lessons in LL-G benefit every repo."
- The `/add-lesson` skill makes contributing to LL-G frictionless
- This practice complements RULE 1 (check LL-G before writing) with the write side (contribute back after learning)
