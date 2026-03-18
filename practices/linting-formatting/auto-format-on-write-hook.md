---
concern: linting-formatting
tech: [claude-code, biome]
priority: recommended
source-repo: 60k-mono
applies-to: [typescript, node]
---
# Auto-Format on Write via PostToolUse Hook

## PATTERN
Configure a Claude Code PostToolUse hook that automatically runs the project's formatter (Biome, Prettier, or ESLint --fix) whenever Claude edits or writes a file. The hook matches Edit and Write tool calls and runs the formatter on the affected file.

## WHY
Without auto-formatting, Claude Code's output may not match the project's style conventions, requiring manual formatting passes. The PostToolUse hook eliminates this friction by formatting automatically after every write, keeping diffs clean and consistent.

## EXAMPLE
From 60k-mono (`.claude/settings.json`):
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx @biomejs/biome check --write ${file_path}",
            "matcher_details": {
              "file_pattern": "\\.(ts|tsx|js|jsx|json|jsonc)$"
            }
          }
        ]
      }
    ]
  }
}
```

For Prettier-based projects:
```json
{
  "command": "npx prettier --write ${file_path}"
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] `.claude/settings.json` exists with a `hooks.PostToolUse` section
- [ ] A hook matches `Edit|Write` tool calls
- [ ] The hook runs the project's formatter on the affected file
- [ ] File pattern filter limits formatting to relevant file types

## IMPLEMENT
1. Ensure the project has a formatter installed (Biome, Prettier, or ESLint)
2. Add PostToolUse hook to `.claude/settings.json` matching `Edit|Write`
3. Set the command to run the formatter with `--write` flag on `${file_path}`
4. Add file pattern filter to avoid formatting non-code files
5. Test by editing a file and verifying it's auto-formatted

## NOTES
- `${file_path}` is a Claude Code hook variable containing the path of the affected file
- This pairs naturally with Biome (see `linting-formatting/biome-replacing-eslint`)
- PostToolUse hooks also fire for Write operations, not just Edit
- tech-assistant uses PostToolUse hooks to enforce PowerShell script rules (no `#Requires -Modules`)
