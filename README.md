# BP -- Best Practices Knowledge Base

A machine-readable knowledge base of **37 proven coding patterns** extracted from 28 repositories. Designed for consumption by Claude Code via `/add-dir C:\Github\BP` and the two-level `llms.txt` index system.

**[View the visual guide](guide.html)** for a human-friendly overview of all practices.

## What This Is

Where [LL-G](https://github.com/wellforce-brandon/LL-G) tracks gotchas and anti-patterns (what NOT to do), BP tracks proven patterns (what TO do). Together they form a complete reference for writing better code with AI assistants.

| | LL-G | BP |
|---|---|---|
| Focus | Gotchas, anti-patterns | Proven patterns, best practices |
| Entry format | PROBLEM / WRONG / RIGHT | PATTERN / WHY / EXAMPLE / CHECK / IMPLEMENT |
| When consulted | Before writing code (RULE 1) | Before starting new work (RULE 3) |
| Severity | HIGH / MEDIUM / LOW | FOUNDATIONAL / RECOMMENDED / OPTIONAL |
| Indexed by | Technology | Concern (with tech tags) |

## Structure

```
practices/
  ai-workflow/          # Plan-then-execute, anti-patterns, parallel sessions, TDD, prompting
  context-management/   # Compaction, handoff docs, subagents, agent memory
  safety/               # Read-only-first, credential deny-list, code review, permissions gate
  claude-config/        # CLAUDE.md structure, rules, hooks, skills, agents, llms.txt
  documentation/        # Reference files, plans with lessons learned, ADRs
  deployment/           # Docker multi-stage, Cloudflare, Doppler secrets
  linting-formatting/   # Biome, auto-format hooks
  versioning/           # Major.Minor.Patch.Build, changelog enforcement
  monorepo/             # pnpm + Turborepo, shared packages with layer deps
  testing/              # Vitest monorepo config, Playwright E2E
  design-systems/       # Guardrail-driven design, OKLCH token architecture
  environment/          # Doppler + .env.local fallback
  error-handling/       # Structured logging with Winston
  knowledge-bases/      # Two-level index pattern for machine-readable KBs
```

14 concern categories. Each contains an `llms.txt` index and individual `.md` entry files with YAML frontmatter.

## Priority Levels

- **FOUNDATIONAL** (10 entries) -- Every repo should follow these. Always loaded.
- **RECOMMENDED** (25 entries) -- Strong patterns for matching tech stacks. Loaded when relevant.
- **OPTIONAL** (2 entries) -- Nice-to-have improvements. Loaded on request.

## Usage

### For AI assistants (RULE 3)
```
Step 1: Read llms.txt (master index)
Step 2: Read the llms.txt for each relevant concern
Step 3: Load all FOUNDATIONAL entries
Step 4: Load RECOMMENDED entries matching your tech tags
```

### Locally
```
/add-dir C:\Github\BP
```

### Via GitHub
```
WebFetch https://raw.githubusercontent.com/wellforce-brandon/BP/main/llms.txt
```

## Skills

| Skill | Purpose |
|-------|---------|
| `/add-practice` | Manually add a new best practice entry |
| `/audit-repo` | Audit a repo against applicable practices, save checklist |
| `/apply-practice <slug>` | Apply a single practice to a target repo |
| `/fix-audit` | Apply all failing practices from the most recent audit |
| `/review-practices` | Review entries for quality/conflicts, search web for new practices |

## Auto-Discovery

Before every git commit, Claude scans the staged changes for novel infrastructure/tooling/config patterns (per RULE 4 in the global CLAUDE.md). If a candidate is found, Claude reads the `practice-scout.md` agent and follows its instructions to create a BP entry. Most commits produce nothing -- only genuinely reusable patterns are captured.

## Repo Inventory

BP tracks 28 repositories with their tech stacks, frameworks, and characteristics in `.claude/references/repo-inventory.md`. The `/audit-repo` skill uses this to determine which practices apply to each repo.

## Self-Maintenance

The `/review-practices` skill handles:
- **Internal review** -- format compliance, conflicts between entries, index integrity
- **External discovery** -- web search for current best practices, gap analysis
- **Inventory refresh** -- scan `C:\Github\` for new/changed repos
- **Deprecation check** -- validate source patterns still exist, mark stale entries

## Integration

Every repo in `C:\Github\` has been rolled out with:
- RULE 3 in CLAUDE.md (check BP before config/tooling work)
- `.claude/rules/bp-check.md` (path-scoped enforcement on config files)
- `.claude/bp-audit.md` (FOUNDATIONAL audit results with actionable checklist)

LL-G's `/add-lesson` skill cross-references BP when creating gotcha entries, suggesting complementary best practice entries when a gotcha implies a positive pattern.

## Contributing

Run `/add-practice` from any session, or the practice scout will auto-discover patterns before commits (RULE 4). For manual additions:

1. Create `practices/<concern>/<slug>.md` using the entry format in CLAUDE.md
2. Append a bullet to `practices/<concern>/llms.txt`
3. Increment the entry count in the master `llms.txt`
