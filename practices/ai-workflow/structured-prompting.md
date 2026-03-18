---
concern: ai-workflow
tech: [claude-code]
priority: recommended
source-repo: claude-code-bootstrap
applies-to: [all]
---
# Structured Prompting for AI Coding Assistants

## PATTERN
When configuring AI coding assistants, provide instructions that are: specific (concrete file paths, exact commands, not vague guidance), scoped (only load what's relevant to the current task), actionable (tell the AI what to DO, not what to KNOW), and verifiable (include checks that confirm the instruction was followed). Avoid generic advice the model already knows natively.

## WHY
AI models already know generic best practices like "write clean code" or "handle errors." Repeating these wastes context and produces no behavioral change. Instructions that change behavior are specific to YOUR codebase, YOUR conventions, and YOUR constraints. Every line in CLAUDE.md should make the AI do something it wouldn't do by default.

## EXAMPLE
Bad (generic, wastes context):
```markdown
- Write clean, readable code
- Use meaningful variable names
- Handle errors properly
```

Good (specific, actionable, verifiable):
```markdown
- All timestamps use `timestamp('...', { withTimezone: true })`
- Never import from `apps/`, only from `packages/`
- Use `@devtools/` namespace with `workspace:*` versions
- Run `biome check --write` after editing .ts files
```

From awesome-copilot instructions pattern:
```markdown
## Instead of:
"Follow best practices for React"

## Write:
"Use shadcn/ui primitives. Max 200 lines per component.
Max 8 props. Boolean props use is*/has* prefix.
Dark mode default for dashboards, light for marketing."
```

## CHECK
- [ ] CLAUDE.md contains project-specific instructions (file paths, commands, naming conventions)
- [ ] CLAUDE.md does NOT contain generic advice the model already follows
- [ ] Instructions include verifiable criteria (file limits, naming patterns, tool commands)
- [ ] CLAUDE.md is under 200 lines (signals pruning of generic content)

## IMPLEMENT
1. Audit CLAUDE.md for generic statements that don't change model behavior
2. Remove or replace generic advice with project-specific instructions
3. Add concrete constraints: file size limits, naming conventions, import rules
4. Include tool commands: what to run, when, with what flags
5. Prune after model updates -- newer models handle more things natively

## NOTES
- "Do NOT bloat CLAUDE.md with generic advice the model already knows" -- Janki CLAUDE.md
- "Prune after every model update -- remove what the model handles natively" -- Janki
- The 200-line CLAUDE.md limit is both a guideline and a forcing function for specificity
- Source: awesome-copilot instructions, Bhartendu-Kumar/rules_template, Janki
