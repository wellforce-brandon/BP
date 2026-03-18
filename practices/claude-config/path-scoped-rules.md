---
concern: claude-config
tech: [claude-code]
priority: recommended
source-repo: MCP
applies-to: [all]
---
# Path-Scoped Rules via .claude/rules/

## PATTERN
Use `.claude/rules/*.md` files with glob-pattern frontmatter to create conditional instructions that only load when Claude is working with matching file paths. This keeps the root CLAUDE.md lean while enforcing domain-specific rules automatically.

## WHY
Rules that only apply to certain files (e.g., "update CHANGELOG.md before every commit" or "check LL-G before writing code") waste context when loaded for unrelated tasks. Path-scoped rules solve this by activating only when relevant files are being edited.

## EXAMPLE
From MCP (`.claude/rules/commit-changelog.md`):
```markdown
# Pre-Commit: Changelog & Version Update

Before every `git commit`, you MUST:

## 1. Update CHANGELOG.md
- Add entries under `[Unreleased]` using Keep a Changelog sections
- Review staged changes (`git diff --cached`) to determine what changed

## 2. Bump Version in package.json
Version format: Major.Minor.Patch.Build
- Build increments on every commit, no exceptions
- When a higher segment increments, all lower segments reset to 0
```

From MCP (`.claude/rules/llg-check.md`):
```markdown
# RULE 1 Enforcement: Check LL-G Before Writing Code

Before writing or editing any file matching the paths above, you MUST
consult the LL-G knowledge base to avoid known failure patterns.
```

## CHECK
How to verify if a repo already follows this:
- [ ] `.claude/rules/` directory exists
- [ ] Rule files have appropriate content for domain-specific concerns
- [ ] Root CLAUDE.md does not contain content that should be path-scoped

## IMPLEMENT
1. Create `.claude/rules/` directory
2. Identify instructions in CLAUDE.md that only apply to certain file types or workflows
3. Extract those into separate `.md` files in `.claude/rules/`
4. Remove the extracted content from root CLAUDE.md

## NOTES
- Common candidates for path-scoped rules: commit/changelog rules, LL-G check enforcement, linting rules for specific file types
- Rules files are loaded automatically by Claude Code when matching paths are in context
