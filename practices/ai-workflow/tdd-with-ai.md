---
concern: ai-workflow
tech: [claude-code, vitest, jest]
priority: recommended
source-repo: bp-website
applies-to: [typescript, node]
---
# Test-Driven Development with AI Assistants

## PATTERN
When using AI coding assistants, write tests BEFORE implementation. Give the AI the test file first and let it implement to pass the tests. This constrains the AI to produce exactly what's needed (not more, not less) and provides immediate verification. Verify tests fail before writing implementation.

## WHY
AI assistants tend to over-engineer when given open-ended implementation tasks. Tests act as a constraint -- the AI knows exactly what "done" looks like. Without tests, the AI may add features you didn't ask for, implement edge cases that don't exist, or produce code that's correct in isolation but doesn't integrate. TDD with AI produces tighter, more predictable output.

## EXAMPLE
From bp-website CLAUDE.md:
```markdown
## Development Principles
3. **TDD Required** -- write tests first, then implement

## Testing
- Write tests before implementation when practical.
- Verify tests fail before writing the implementation.
- Run tests after every change.
- Prefer integration tests for behavior, unit tests for logic.
```

Workflow:
```
1. Write test: "ContactForm submits to /api/contact with name and email"
2. Run test -- verify it FAILS (no implementation yet)
3. Ask AI: "Implement ContactForm to pass this test"
4. Run test -- verify it PASSES
5. Repeat for next behavior
```

## CHECK
- [ ] CLAUDE.md mentions writing tests before implementation
- [ ] CLAUDE.md mentions verifying tests fail first
- [ ] Test runner is configured and working (vitest, jest, etc.)
- [ ] Integration tests exist for core behaviors

## IMPLEMENT
1. Add TDD guidance to CLAUDE.md testing section
2. Include the "verify tests fail first" step explicitly
3. Prefer integration tests for AI-generated code (they catch more integration issues)
4. Set up test runner if not already configured

## NOTES
- Per-package contract tests verify shared package APIs in monorepos
- Integration tests catch more real-world issues than unit tests with AI-generated code
- "Prefer integration tests for behavior, unit tests for logic" -- consistent across all repos
- Source: bp-website, awesome-copilot testing instructions, NikiforovAll/claude-code-rules Playwright plugin
