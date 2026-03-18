---
concern: claude-config
tech: [claude-code, markdown]
priority: recommended
source-repo: BP
applies-to: [all]
---
# llms.txt as a Standard Project Artifact

## PATTERN
Create an `llms.txt` file at the repo root following the llms.txt convention -- a machine-readable summary of the project's purpose, structure, and key files that any LLM can quickly parse. This is distinct from CLAUDE.md (which contains instructions) -- llms.txt is a table of contents for the project itself.

## WHY
LLMs waste significant tokens reading files to understand a project. An llms.txt provides a structured entry point: what the project is, how it's organized, and where to find key files. This is especially valuable for repos that are frequently loaded via `/add-dir` or consumed by external tools. The convention is recognized by LLMs as a machine-readable index.

## EXAMPLE
```markdown
# ProjectName

> One-line description of what this project does.

## Key Files

- [CLAUDE.md](CLAUDE.md): AI assistant instructions
- [README.md](README.md): Human-readable project overview
- [package.json](package.json): Dependencies and scripts

## Architecture

- [src/api/](src/api/): API routes and controllers
- [src/components/](src/components/): React UI components
- [src/lib/](src/lib/): Shared utilities and helpers
- [packages/](packages/): Shared monorepo packages

## Documentation

- [docs/architecture.md](docs/architecture.md): System architecture
- [.claude/references/](claude/references/): AI-readable reference docs
```

## CHECK
- [ ] `llms.txt` exists at repo root
- [ ] File starts with project name as H1 and one-line description as blockquote
- [ ] Key files and directories are listed with relative links
- [ ] File is under 50 lines (index, not documentation)

## IMPLEMENT
1. Create `llms.txt` at repo root
2. Add project name, one-line description, and key file links
3. Organize by: Key Files, Architecture, Documentation
4. Keep under 50 lines -- this is a table of contents, not a manual

## NOTES
- Source: AnswerDotAI/llms-txt spec, awesome-copilot create-llms skill, thedaviddias/llms-txt-hub
- llms.txt is for project discovery; CLAUDE.md is for behavior instructions
- The llms.txt convention is at https://llmstxt.org/
- BP and LL-G both use llms.txt internally -- this practice makes it standard for all repos
