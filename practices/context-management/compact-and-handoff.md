---
concern: context-management
tech: [claude-code]
priority: foundational
source-repo: 60k-mono
applies-to: [all]
---
# Proactive Compaction and Session Handoff

## PATTERN
Use `/compact` proactively at ~50% context usage, not when forced by the limit. For multi-session work, create a HANDOFF.md file in `tasks/` before ending a session that captures: current state, what's done, what's next, key decisions made, and files touched. The next session loads HANDOFF.md as its sole starting context.

## WHY
Context compaction at the limit loses nuance and recent decisions. Proactive compaction preserves more signal. Without handoff docs, the next session starts from scratch -- re-reading files, re-discovering state, and potentially contradicting earlier decisions. A structured handoff makes multi-session work feel continuous.

## EXAMPLE
From 60k-mono/Janki CLAUDE.md:
```markdown
## Context Management
- Keep CLAUDE.md under 200 lines for reliable adherence.
- Break tasks small enough to complete in under 50% context usage.
- Use `/compact` proactively around 50% context.
- Start fresh conversations for unrelated topics.
- Begin complex tasks in plan mode before implementation.
- Document failed attempts to `.claude/agent-memory/debugging.md`
  before starting new sessions. Avoids repeating dead ends.
- Use `/handoff` to create a summary before ending a session.
  Load in fresh session as sole context.
```

Handoff doc structure:
```markdown
# Handoff: [Task Name]
Date: 2026-03-18

## Status
[What's done, what's in progress]

## Key Decisions
[Decisions made this session with rationale]

## Files Changed
[List of modified files with brief description]

## Next Steps
1. [Specific next action]
2. [Following action]

## Blockers / Open Questions
[Anything unresolved]
```

## CHECK
- [ ] CLAUDE.md mentions `/compact` at 50% or similar proactive threshold
- [ ] CLAUDE.md mentions handoff docs for multi-session work
- [ ] `tasks/` directory exists for plan and handoff files

## IMPLEMENT
1. Add context management section to CLAUDE.md with compact and handoff guidance
2. Create `tasks/` directory if missing
3. Add handoff doc template to `.claude/references/` or CLAUDE.md

## NOTES
- "Code bias fix" pattern: if stuck in bad patterns, build the feature in isolation in a fresh folder, then port it in
- Failed attempts should be documented in `.claude/agent-memory/debugging.md` to prevent repeating dead ends across sessions
- Source: ykdojo/claude-code-tips, Janki CLAUDE.md, 60k-mono CLAUDE.md
