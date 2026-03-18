---
concern: claude-config
tech: [claude-code]
priority: recommended
source-repo: Janki
applies-to: [all]
---
# Skill Frontmatter Configuration

## PATTERN
Skills in `.claude/skills/<name>/SKILL.md` use YAML frontmatter to control execution behavior: which model runs the skill, whether it runs in an isolated context, and whether it auto-invokes or requires manual trigger.

Key frontmatter fields:
- `name` -- skill identifier (used for `/skillname` invocation)
- `description` -- what the skill does (used for auto-matching)
- `model: haiku|sonnet|opus` -- which model runs the skill (haiku for step-by-step, sonnet for analysis, opus for planning/orchestration)
- `disable-model-invocation: true` -- prevents auto-loading; must use `/skillname`
- `context: fork` -- runs in isolated subagent context (prevents context contamination)
- `agent: <name>` -- binds execution to a specific agent's persona and tools

## WHY
Without explicit model selection, every skill runs on whatever model the session is using -- often overkill for simple tasks (wasting tokens) or underpowered for complex ones (producing poor results). Frontmatter lets you right-size each skill.

## EXAMPLE
From LL-G (`.claude/skills/add-lesson/SKILL.md`):
```yaml
---
name: add-lesson
description: Add a new gotcha or lesson learned to the LL-G knowledge base
model: haiku
---
```

From Janki (`.claude/skills/spec-developer/SKILL.md`):
```yaml
---
name: spec-developer
description: Interview-driven feature spec saved to /tasks
model: opus
---
```

## CHECK
How to verify if a repo already follows this:
- [ ] Skills exist in `.claude/skills/<name>/SKILL.md` format
- [ ] Each skill has YAML frontmatter with at least `name` and `description`
- [ ] Model is specified based on skill complexity (haiku for simple, sonnet for medium, opus for complex)

## IMPLEMENT
1. Create `.claude/skills/` directory if it doesn't exist
2. For each skill, create `<skill-name>/SKILL.md` with frontmatter
3. Choose appropriate model: haiku for data collection/formatting, sonnet for analysis/review, opus for planning/orchestration
4. Add a skills table to CLAUDE.md listing all available skills

## NOTES
- `${CLAUDE_SKILL_DIR}` variable references the skill's own directory for relative file access
- Agent frontmatter supports additional fields: `background: true`, `isolation: worktree`, `maxTurns: N`
- List skills in a table in CLAUDE.md so users know what's available
