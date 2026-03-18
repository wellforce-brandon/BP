---
concern: linting-formatting
tech: [biome, typescript]
priority: recommended
source-repo: 60k-mono
applies-to: [typescript, node]
---
# Biome as Unified Linter and Formatter

## PATTERN
Use Biome as a single tool replacing both ESLint and Prettier. Biome handles linting, formatting, and import sorting in one pass with significantly faster execution. Configure via `biome.json` or `biome.jsonc` at the repo root.

## WHY
Running ESLint + Prettier separately is slow (especially in monorepos), requires coordinating two configs, and often produces conflicts between the two tools. Biome is 10-100x faster (Rust-based), provides both linting and formatting in a single tool, and eliminates the ESLint/Prettier conflict class entirely.

## EXAMPLE
From 60k-mono -- PostToolUse hook for auto-formatting on save:
```json
{
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
```

Biome config (`biome.jsonc`):
```jsonc
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  }
}
```

## CHECK
How to verify if a repo already follows this:
- [ ] `@biomejs/biome` is in devDependencies
- [ ] `biome.json` or `biome.jsonc` exists at repo root
- [ ] No ESLint + Prettier combination in devDependencies (or migration planned)
- [ ] Lint/format scripts use `biome check` or `biome format`

## IMPLEMENT
1. Install Biome: `pnpm add -D @biomejs/biome` (or npm equivalent)
2. Create `biome.jsonc` with recommended rules, import sorting, and formatting config
3. Update package.json scripts: `"lint": "biome check .", "format": "biome format --write ."`
4. Remove ESLint and Prettier if migrating: uninstall packages, delete configs
5. Add PostToolUse hook in `.claude/settings.json` for auto-format on write

## NOTES
- Biome 2.x is current as of 2026; verify latest version before adopting
- Some ESLint plugins (e.g., eslint-plugin-react-hooks) have Biome equivalents, check compatibility
- Repos using Next.js may keep `next lint` alongside Biome for Next.js-specific rules
- The PostToolUse hook makes auto-formatting transparent -- Claude Code formats on every edit
