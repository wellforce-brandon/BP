---
concern: ai-workflow
tech: [claude-code]
priority: foundational
source-repo: Janki
applies-to: [all]
---
# Plan in One Session, Execute in Another

## PATTERN
For non-trivial work, always plan in one session and execute in a fresh session. The planning session researches, evaluates options, and produces a concrete plan saved to `tasks/`. The execution session loads only the plan and implements it. Never plan and execute in the same context window for complex features.

## WHY
Planning pollutes context with research, comparisons, and discarded options. By the time you start implementing, 50%+ of context is spent on exploration that's no longer relevant. A fresh session with just the plan has maximum context available for the actual implementation. This also forces the plan to be complete enough to stand alone.

## EXAMPLE
From Janki CLAUDE.md:
```markdown
## Workflow: Plan First, Then Init
1. Say "plan repo" to choose stack, generate README, create guardrails.
2. Say "initialize repo" to configure Claude Code using the plan.
3. Say "update practices" periodically to stay current.

For features: say "spec developer" to generate a detailed plan,
then start a fresh session to implement it.
```

From 60k-mono:
```markdown
## Planning
- Planning is phase-based, not timeline-based.
  Phases: Foundation, Core, Polish, Ship.
- Always plan in one session, execute in another.
- Save every plan to a /tasks folder.
- For big features, use the spec-developer skill.
```

## CHECK
- [ ] CLAUDE.md states "plan in one session, execute in another" or equivalent
- [ ] `tasks/` directory exists for storing plans
- [ ] A spec-developer or equivalent planning skill is available

## IMPLEMENT
1. Add planning guidance to CLAUDE.md
2. Create `tasks/` directory
3. Set `"plansDirectory": "tasks"` in `.claude/settings.json`
4. Ensure plans are saved as markdown files, not just in conversation

## NOTES
- Phase-based planning (Foundation, Core, Polish, Ship) works better than timeline-based
- The `/spec-developer` skill does interview-driven spec generation with 20+ clarifying questions
- Every plan must end with Lessons Learned section (see documentation/plan-with-lessons-learned)
- Source: Janki, 60k-mono, awesome-copilot agent patterns
