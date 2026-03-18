---
concern: documentation
tech: [claude-code]
priority: recommended
source-repo: 60k-mono
applies-to: [all]
---
# Reference Files in .claude/references/

## PATTERN
Store operational reference documents in `.claude/references/` and link them from CLAUDE.md with a "Read Before" table. Reference files contain stable knowledge that Claude needs when working in specific areas -- infrastructure definitions, API patterns, design guardrails, tool registries, source URLs.

## WHY
Putting all reference material in CLAUDE.md bloats it past the adherence threshold. Reference files keep CLAUDE.md lean (under 200 lines) while still providing deep context on demand. The "Read Before" table tells Claude exactly when to load each reference.

## EXAMPLE
From 60k-mono (`CLAUDE.md`):
```markdown
## Key Documentation

| Document | Purpose |
|----------|---------|
| `.claude/references/design-guardrails.md` | UI rules, accessibility, performance budgets |
| `.claude/references/tools.md` | CLI tools, MCP servers, install commands |
| `.claude/references/infrastructure.md` | Hosting stack (locked, do not modify per-project) |
| `.claude/references/source-urls.md` | URLs for fetching best practices |
```

From tech-assistant (`CLAUDE.md`):
```markdown
## Operational Reference Files

| File | Read Before |
|------|-------------|
| `deploy/DEPLOYMENT-PATTERNS.md` | Modifying deployment scripts |
| `.claude/NINJAONE-API-PATTERNS.md` | Any NinjaOne skill or automation |
```

## CHECK
How to verify if a repo already follows this:
- [ ] `.claude/references/` directory exists
- [ ] At least one reference file exists (tools.md, source-urls.md, etc.)
- [ ] CLAUDE.md contains a documentation table linking to reference files
- [ ] Reference files are stable (not frequently changing)

## IMPLEMENT
1. Create `.claude/references/` directory
2. Create `source-urls.md` with URLs for fetching best practices (never hardcode URLs in skills)
3. Create `tools.md` listing CLI tools, MCP servers, and install commands used in the project
4. Add a documentation table to CLAUDE.md linking each reference file with its "Read Before" context
5. Move any stable reference content from CLAUDE.md into reference files

## NOTES
- Common reference files: `source-urls.md`, `tools.md`, `infrastructure.md`, `design-guardrails.md`
- Skills should reference `source-urls.md` for URLs instead of hardcoding them
- Infrastructure reference should be marked as locked/immutable if it defines fixed platform choices
