---
concern: knowledge-bases
tech: [claude-code, markdown]
priority: foundational
source-repo: LL-G
applies-to: [all]
---
# Two-Level Index Pattern for Machine-Readable KBs

## PATTERN
Use a two-level `llms.txt` index system for knowledge bases consumed by LLMs. A master `llms.txt` at the repo root lists all categories with links to per-category `llms.txt` files. Each category index lists individual entries with one-line descriptions and severity/priority tags. This lets the LLM load only the relevant subset.

## WHY
Loading an entire knowledge base into context is wasteful and often exceeds context limits. The two-level index pattern gives the LLM a "table of contents" that it can navigate selectively -- load the master index, identify relevant categories, load only those category indexes, then load only the specific entries it needs.

## EXAMPLE
From LL-G -- Master `llms.txt`:
```markdown
# LL-G Lessons Learned & Gotchas

> Machine-readable knowledge base of coding gotchas. Before scripting,
> load this file, then load the llms.txt for each technology you are
> about to use.

## Technologies

### PowerShell
- [PowerShell index](kb/powershell/llms.txt): All PowerShell gotchas (12 entries)

### Graph API (Microsoft)
- [Graph API index](kb/graph-api/llms.txt): All Graph API gotchas (17 entries)
```

Per-category `kb/powershell/llms.txt`:
```markdown
# PowerShell Gotchas

> Known PowerShell patterns that cause silent failures.

## Entries

- [Variable quoting in strings](quoting.md): Double quotes expand
  variables; single quotes are literal. HIGH.
- [Error handling with -ErrorAction Stop](error-handling.md): Default
  error action is Continue -- errors silently swallowed. HIGH.
- [Array safety with @() wrapping](array-safety.md): Single pipeline
  result becomes scalar, not array. HIGH.
```

## CHECK
How to verify if a repo already follows this:
- [ ] Root `llms.txt` exists with category listing
- [ ] Each category has its own `llms.txt` with entry listing
- [ ] Entries include one-line descriptions and severity/priority tags
- [ ] Links are relative paths that resolve correctly

## IMPLEMENT
1. Create root `llms.txt` with repo description, usage instructions, and category listing
2. For each category, create a `<category>/llms.txt` with entry descriptions
3. Use relative paths for all links
4. Include severity/priority tag on each entry line (e.g., "HIGH.", "FOUNDATIONAL.")
5. Include entry counts in the master index: `(N entries)`

## NOTES
- The `llms.txt` convention is recognized by LLMs as a machine-readable index
- Entry counts in the master index help the LLM gauge how much context a category will consume
- This pattern is used by both LL-G (gotchas) and BP (best practices)
- Keep descriptions to one line so the index itself is scannable without loading entries
