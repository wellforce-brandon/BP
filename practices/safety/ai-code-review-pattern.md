---
concern: safety
tech: [claude-code]
priority: recommended
source-repo: 60k-mono
applies-to: [all]
---
# AI-Assisted Code Review with Severity Levels

## PATTERN
Use a dedicated reviewer agent (or `/code-review` skill) that runs a structured checklist: correctness, naming, DRY, error handling, type safety, test coverage, dead code, TODO/FIXME/HACK markers, standards compliance, security, and performance. Findings are categorized as CRITICAL (must fix), WARNING (should fix), or SUGGESTION (nice to have). Output format: `[SEVERITY] file:line -- description + recommendation`.

## WHY
Manual code review is slow and inconsistent. AI review agents apply the same checklist every time, catch mechanical issues humans miss (unused imports, missing error handling, type safety gaps), and free up human reviewers to focus on architecture and business logic. The severity system prevents alert fatigue by distinguishing blocking issues from suggestions.

## EXAMPLE
From 60k-mono (`.claude/agents/reviewer.md`):
```yaml
name: reviewer
description: Code review focused on correctness, maintainability
model: sonnet
permissionMode: plan
maxTurns: 20
tools: [Read, Glob, Grep]
```

Review checklist: Correctness, Naming, DRY, Error Handling, Type Safety, Test Coverage, Dead Code, Standards Compliance, Security, Performance.

Output format:
```
[CRITICAL] src/auth/session.ts:45 -- Session token stored in
localStorage (XSS risk). Recommendation: Use httpOnly cookie.

[WARNING] src/api/users.ts:112 -- Missing error handling on
database query. Recommendation: Wrap in try/catch with proper
error response.

[SUGGESTION] src/utils/format.ts:8 -- Function could be simplified
with optional chaining. Recommendation: Use user?.name ?? 'Unknown'.
```

## CHECK
- [ ] A `/code-review` skill or reviewer agent exists
- [ ] Review uses a structured checklist (not ad-hoc)
- [ ] Findings are categorized by severity (CRITICAL/WARNING/SUGGESTION)
- [ ] Output includes file:line references

## IMPLEMENT
1. Create `.claude/agents/reviewer.md` with the checklist and output format
2. Create `.claude/skills/code-review/SKILL.md` as the user-invocable trigger
3. Set model to sonnet (good balance of speed and depth)
4. Set `permissionMode: plan` (reviewer should read, not write)

## NOTES
- Run review before committing, not after
- Distinguish blocking issues (CRITICAL) from suggestions to prevent alert fatigue
- The reviewer agent should "not nitpick style if it matches conventions"
- Source: 60k-mono reviewer agent, awesome-copilot code review instructions, NikiforovAll/claude-code-rules
