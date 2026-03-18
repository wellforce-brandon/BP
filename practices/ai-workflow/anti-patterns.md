---
concern: ai-workflow
tech: [claude-code]
priority: foundational
source-repo: external
applies-to: [all]
---
# AI Session Anti-Patterns and Recovery

## PATTERN
Recognize three common failure modes in AI coding sessions and apply the correct recovery:

1. **Correction Loop** -- Correcting the AI more than twice on the same issue. Context fills with failed attempts, making each subsequent attempt worse. Recovery: `/clear` and write a better initial prompt incorporating what you learned.

2. **Kitchen Sink Session** -- Starting with one task, asking something unrelated, then going back. Context fills with irrelevant noise. Recovery: `/clear` between unrelated tasks, or start a fresh session.

3. **Infinite Exploration** -- Unscoped "investigate" or "look into" prompts cause the AI to read hundreds of files. Recovery: scope narrowly ("check src/auth/session.ts for token expiry logic") or delegate to a subagent.

## WHY
These anti-patterns waste tokens, degrade output quality, and create frustration. Context window quality matters more than quantity -- a 50% full window of focused context outperforms a 90% full window of mixed context. Recognizing the pattern early and recovering (usually via `/clear`) saves significant time.

## EXAMPLE
Bad (correction loop):
```
You: "Make the button blue"
AI: [makes it teal]
You: "No, blue, like #0066FF"
AI: [makes it blue but breaks hover state]
You: "The hover is broken now, fix it"
AI: [fixes hover but changes the color again]
-- context is now 30% correction history --
```

Good (clear and re-prompt):
```
You: "Make the button blue"
AI: [makes it teal]
You: /clear
You: "Change the button in src/components/Button.tsx to
background-color: #0066FF. Keep the existing hover state
(opacity: 0.8) unchanged."
AI: [correct on first try]
```

## CHECK
- [ ] CLAUDE.md mentions `/clear` for unrelated tasks or stuck loops
- [ ] Team knows the three anti-patterns and their recovery actions
- [ ] Context management section exists in CLAUDE.md

## IMPLEMENT
1. Add anti-pattern awareness to CLAUDE.md context management section
2. Include the recovery actions: `/clear` for loops and kitchen sinks, subagent for exploration
3. Set the "two corrections max" rule -- if stuck twice, clear and re-prompt

## NOTES
- Source: Anthropic official best practices, shanraisshan/claude-code-best-practice
- The `/btw` command is useful for side questions that don't need to persist in context
- Session naming with `/rename` helps track which sessions are for which workstream
- "If Claude ignores your rules, the CLAUDE.md file is too long, not too short"
